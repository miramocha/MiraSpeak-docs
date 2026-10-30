# Remote MCP access with Cloudflare Tunnel + Access

MiraSpeak's MCP server only listens on `127.0.0.1`. To let a cloud AI agent (for example a hosted assistant) drive MiraSpeak on your PC, publish it through a **Cloudflare Tunnel** and lock it with **Cloudflare Access** using a service token.

- No router ports opened; `cloudflared` only makes outbound connections, and your home IP stays hidden.
- Without the service token, every request gets a Cloudflare `403` before it reaches your PC.
- Cost: Tunnel and Zero Trust Free (up to 50 users) are free. You need a domain whose DNS is on Cloudflare (it can stay registered elsewhere; just point its nameservers at Cloudflare).

> **Security note:** whoever holds the token can call every MCP tool, including file-reading ones like `speak_from_file`. Keep the token secret, never show it on stream, and turn MCP off when you don't need it.

## 1. Enable MCP in MiraSpeak

Turn on the MCP server in MiraSpeak's settings. Default endpoint: `http://127.0.0.1:56769/mcp` (port is `mcp_port` in `settings.toml`).

Keep MCP **off** until step 4 is done, so the URL is never exposed without Access.

## 2. Create the tunnel

1. In the [Zero Trust dashboard](https://one.dash.cloudflare.com/): **Networks → Tunnels → Create a tunnel**, pick **Cloudflared**, name it (e.g. `miraspeak`).
2. Choose your OS (Windows **64-bit** for most PCs) and run the shown install command in an admin terminal. That command contains a token: keep it private.
3. Wait until the tunnel shows **Healthy**.

## 3. Add a route

On the tunnel, **Add route → Published application**:

| Field | Value |
| --- | --- |
| Subdomain | something specific, e.g. `miraspeak-mcp-dev` (keep plain names for product pages) |
| Domain | your Cloudflare domain |
| Path | *(empty)* |
| Service URL | `http://localhost:56769` (plain `http`) |
| Additional settings → HTTP Settings → **HTTP Host Header** | `localhost:56769` |

The Host Header override is required: the MCP library rejects requests whose `Host` isn't localhost.

## 4. Protect it with Access

1. Pick the **Zero Trust Free** plan if asked.
2. **Access → Service credentials → Service Tokens → Add a service token.** Name it (e.g. `grokbot`), set a duration (1 year is fine). Copy the **Client ID** and **Client Secret** immediately: the secret is shown only once. If you miss it, refresh or recreate the token.
3. **Access → Applications → Add an application → Self-hosted.** Add the same public hostname, Path empty.
4. Create a policy:
   - Action: **Service Auth** (not Allow)
   - Include: Selector **Service Token**, value your token
   - No Require/Exclude rows (delete empty ones)
5. Make sure the policy is actually **attached** to the application (the app must not say "No policy associated"), then save.

### Verify

```sh
# Without the token: expect 403
curl -s -o /dev/null -w "%{http_code}\n" -X POST https://<host>/mcp

# With the token (MCP on): expect 200 and an initialize result naming "MiraSpeak"
curl -s -X POST https://<host>/mcp \
  -H "CF-Access-Client-Id: $CF_ACCESS_CLIENT_ID" \
  -H "CF-Access-Client-Secret: $CF_ACCESS_CLIENT_SECRET" \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -d '{"jsonrpc":"2.0","id":1,"method":"initialize","params":{"protocolVersion":"2025-03-26","capabilities":{},"clientInfo":{"name":"test","version":"1"}}}'
```

A `502` with the token means the tunnel works but MiraSpeak's MCP server isn't running.

## 5. Connect your agent

Add the URL `https://<host>/mcp` as a custom MCP connector and send the two Access headers on every request. If the client can't set headers directly, wrap it with [`mcp-remote`](https://www.npmjs.com/package/mcp-remote), reading the values from environment variables:

```sh
npx -y mcp-remote https://<host>/mcp --transport http-only \
  --header "CF-Access-Client-Id:${CF_ACCESS_CLIENT_ID}" \
  --header "CF-Access-Client-Secret:${CF_ACCESS_CLIENT_SECRET}"
```

Tip: give the agent a rule to call `get_status` before `speak` or `render` and ask first when the active engine is a paid cloud one (Azure, etc.) rather than a local one like Kokoro.

## 6. Get audio files back (`render`)

`render` renders a line to an audio file without playing it on your PC (the Speak page shows it as an **MCP render** row that ends as **Rendered**). It is the remote twin of the `/render` slash command, which saves to a file you pick on the PC instead. The reply never carries the audio itself. It holds a relative `path`:

```json
{"ok": true, "path": "/renders/<token>.mp3", "format": "mp3", "mime_type": "audio/mpeg", "duration_s": 2.48, "bytes": 39821, "expires_in_s": 600}
```

Download it from the same host with the same two Access headers, before it expires (10 minutes; files are also cleared when MCP is turned off or on):

```sh
curl -s -o line.mp3 "https://<host>/renders/<token>.mp3" \
  -H "CF-Access-Client-Id: $CF_ACCESS_CLIENT_ID" \
  -H "CF-Access-Client-Secret: $CF_ACCESS_CLIENT_SECRET"
```

Because the Access application covers the whole hostname (Path empty), `/renders/` is protected by the same service token as `/mcp`. A `404` means the token in the path is unknown or expired. A render waits in the normal speaking queue and is stopped after 90 seconds (under Cloudflare's 100-second limit), so keep long lines short or split them.

## Troubleshooting

| Symptom | Cause |
| --- | --- |
| `502` without token | Access not active yet (policy not attached, or still propagating) and MCP off |
| `403` with token | Wrong token, policy action not **Service Auth**, or token not in Include |
| `502` with token | MCP off in MiraSpeak, or wrong port |
| `421` / host errors | HTTP Host Header not set to `localhost:<port>` |
| `404` on `/renders/…` | Render expired (10 min), MCP was restarted, or the path was mistyped |
