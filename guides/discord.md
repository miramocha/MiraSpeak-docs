# Discord voice channel setup

MiraSpeak's **Discord** module lets a bot join your voice channel and play everything MiraSpeak speaks, so friends hear your TTS voice in calls without a virtual audio cable. This guide creates your own Discord bot and then connects it in MiraSpeak.

- It's **your own bot**. MiraSpeak doesn't ship a shared bot, so the name and avatar are yours to choose.
- It's free. Discord doesn't charge for bots or voice, and the bot runs inside MiraSpeak on your computer.
- MiraSpeak keeps the bot token in your OS credential store (keyring), never in `settings.toml`.

> **Security note:** the bot token lets anyone control your bot. Never commit it, paste it in chat, or show it on stream. If it leaks, reset it (see [Troubleshooting](#troubleshooting)).

## 1. Create the bot in the Discord developer portal

Open the [Discord developer portal](https://discord.com/developers/applications).

1. Click **New Application** and give it a name, for example the name of your character.
2. **Bot** tab: click **Reset Token** and copy the token. It's shown **once**.
   - You need the **Bot token**. The Public Key, Client ID and Client Secret won't work.
3. **Installation** tab: set **Install Link** to **None**.

## 2. Enable the module in MiraSpeak

**Settings → Integrations → Modules**

1. Enable **Discord**.
2. Approve the permissions it asks for.

## 3. Connect the bot

Open the **Discord** page under **Speak**.

1. Paste the token into **Bot token**. It stays private on your computer.
2. Click **Open invite link**, pick your server and click **Authorize**. You need the **Manage Server** permission on that server.
3. Click **Connect**. The status should read `Connected as <bot name>`.
4. Click **Refresh servers**, then pick your **server** and **voice channel**.
5. Click **Join**, then **Test sound**.

From now on, every line MiraSpeak speaks also plays in the voice channel.

## Page settings

| Setting | What it does |
| --- | --- |
| Connect and join when MiraSpeak starts | Reconnects and rejoins the last channel automatically |
| Leave when idle (minutes) | Leaves the channel after this many quiet minutes. `0` means never |

## Troubleshooting

| Symptom | Fix |
| --- | --- |
| "Not in any server" | The invite didn't finish. Click **Open invite link** again, then **Refresh servers** |
| Bot connected but silent | In Discord, right-click the bot and untick **Server Mute** |
| Bot connected but silent | Check that the channel allows the bot to **Speak** |
| Bot connected but silent | Make sure other people are in the voice channel |
| "Discord rejected the token" | **Reset Token** in the portal and paste the new token. Never share it |
| No audio on Linux | Install libopus, for example `sudo apt install libopus0` |
