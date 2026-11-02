# Discord voice channel setup

MiraSpeak's **Discord** module lets a bot join your voice channel and play everything MiraSpeak speaks, so friends hear your TTS voice in calls without a virtual audio cable. This guide creates your own Discord bot and then connects it in MiraSpeak.

- It's **your own bot**. MiraSpeak doesn't ship a shared bot, so the name and avatar are yours to choose.
- It's free. Discord doesn't charge for bots or voice, and the bot runs inside MiraSpeak on your computer.
- MiraSpeak keeps the bot token in your OS credential store (keyring), never in `settings.toml`.

> **Security note:** the bot token lets anyone control your bot. Never commit it, paste it in chat, or show it on stream. If it leaks, reset it (see [Troubleshooting](#troubleshooting)).

## 1. Create the bot in the Discord developer portal

Open the [Discord developer portal](https://discord.com/developers/applications). You can also click **Open developer portal** on MiraSpeak's Discord page.

1. Click **New Application** and give it a name, for example the name of your character.
2. **Bot** tab: click **Reset Token** and copy the token. It's shown **once**.
   - You need the **Bot token**. The Public Key, Client ID and Client Secret won't work.
   - Keep **Public Bot** off, so only you can add it to servers.
3. **Installation** tab: set **Install Link** to **None**.

## 2. Enable the module in MiraSpeak

**Settings → Integrations → Modules**

1. Enable **Discord**.
2. Approve the permissions it asks for.

## 3. Connect the bot

Open the **Discord** page under **Speak**.

**Bot** section

1. Paste the token into **Bot token**. It stays private on your computer.
2. Click **Open invite link**, pick your server and click **Authorize**. You need the **Manage Server** permission on that server.
3. Click **Connect**. The status should read `Connected as <bot name>`.
**Voice channel** section

4. Click **Refresh servers**, then pick your **Server** and **Voice channel**.
5. Join that voice channel yourself in Discord, so you can hear the bot.
6. Click **Join**, then **Test sound**. The status shows how much audio was sent and its peak level.

From now on, every line MiraSpeak speaks also plays in the voice channel. Click **Leave** to disconnect the bot from the channel.

## Behavior settings

| Setting | What it does |
| --- | --- |
| Connect and join when MiraSpeak starts | Reconnects and rejoins the last channel automatically (on by default) |
| Leave when idle | Leaves the channel after this many quiet minutes (default 10, up to 240). `0` means never |

## Troubleshooting

The status line on the Discord page tells you what's wrong. The common messages are below.

| Status message | Fix |
| --- | --- |
| "Connected as …, but the bot is not in any server" | The invite didn't finish. Click **Open invite link** again, then **Refresh servers** |
| "That does not look like a bot token" | You copied the Public Key, Client ID or Client Secret. Copy the token from the **Bot** tab |
| "Discord rejected the token" | **Reset Token** in the portal and paste the new token. Never share it |
| "The bot can't see #…" / "lacks Connect" / "lacks Speak" | Give the bot **View Channel**, **Connect** and **Speak** in that channel's permissions |
| "The bot is server-muted in #…" | In Discord, right-click the bot and untick **Server Mute** |
| "The bot is suppressed in Stage #…" | In a Stage channel, a moderator must invite the bot to speak |
| "The bot muted itself in #…" | Click **Leave**, then **Join** again |
| "Removed from #…" | Someone disconnected the bot. Click **Join** to return |
| "That channel is gone" | Click **Connect** to refresh the lists |
| Test sound plays but nobody hears it | Make sure you or other people are in the voice channel before the bot joins, then click **Leave** and **Join** |
| "Discord voice needs libopus" (Linux) | Install libopus, for example `sudo apt install libopus0` |
| "Discord support is not installed in this copy of MiraSpeak" | Your build doesn't include Discord support. Update to the latest MiraSpeak build |
