# OpenClaw + Telegram Post-Deployment Reference Manual (for humans)

> Version v1.1.0 · OpenClaw 2026.10.8 · Windows 10/11 · DeepSeek
> This manual is for **people**, to refer to *after* deployment. Every command below can be run in any PowerShell window — you do **not** need to `cd` anywhere first.

---

## 0. Three things to remember

1. **No `cd` needed**: `openclaw` is a global command and works from any directory (unlike Claude Code, which you run inside a project folder).
2. **Everything lives in `C:\Users\<you>\.openclaw\`**: config, keys, the agent workspace, and memory are all in this one folder.
3. **Closing the console ≠ stopping the bot**: closing the browser/terminal does not stop the bot; it keeps responding in the background. To actually stop it, run `openclaw daemon stop`.

---

## 1. Location quick reference

| What you're looking for | Location |
|---|---|
| Config file | `C:\Users\<you>\.openclaw\openclaw.json` |
| API keys / credentials / state | `C:\Users\<you>\.openclaw\state\openclaw.sqlite` |
| Agent workspace (where it reads/writes files) | `C:\Users\<you>\.openclaw\workspace\` |
| Agent memory / sessions | `C:\Users\<you>\.openclaw\agents\main\agent\openclaw-agent.sqlite` |
| Logs | `C:\Users\<you>\.openclaw\logs\` |

> Replace `<you>` with your Windows username. Quickest way to find it: run `echo $env:USERPROFILE` in PowerShell.

### Determine / limit the agent workspace

- **Determine**: `openclaw config get agents.defaults.workspace` — if it reports "unset, runtime default applies", the default is `~\.openclaw\workspace\`.
- **Limit**: by default the agent only works inside `~\.openclaw\workspace\` and does not touch your other files. To relocate it: `openclaw config set agents.defaults.workspace "D:\some\folder"` then `openclaw gateway restart`. **Do not put sensitive files into `workspace\`** — the agent can read them.

---

## 2. Daily operations

| What you want to do | Command / how |
|---|---|
| Check status (do this first) | `openclaw status` |
| **Start** the bot | `openclaw daemon start` |
| **Stop** the bot | `openclaw daemon stop` |
| Restart after config changes | `openclaw gateway restart` |
| Watch live logs (troubleshooting) | `openclaw logs --follow` (Ctrl+C to exit) |
| Test the model | `openclaw agent --message "hi"` |
| Open the console (web) | `openclaw dashboard` |
| Open the console (terminal) | `openclaw tui` |
| Close the console | close the browser tab / Ctrl+C in the TUI |

> **After a reboot**: sign in to Windows, wait 1–2 minutes, then `openclaw status` should show the Gateway `running` and Telegram `ON / OK`. If not, run `openclaw daemon start`.

---

## 3. Acceptance checklist (what "done" means)

Send the bot a question that requires some thought in Telegram (e.g. "Explain why the sky is blue, and give three keywords"), then confirm:

1. ✅ It **replies normally** (not "Couldn't sign in…").
2. ✅ You see **only the final answer** — no thinking, reasoning, or tool-call output.
3. ✅ You can **quickly open/close the console** (`openclaw dashboard` → close the tab).

---

## 4. Notes for beginners

- 🔑 **Never leak keys/tokens**: your DeepSeek key and bot token are passwords; don't paste them into public repos, group chats, or screenshots.
- 👥 **Allowlist only yourself**: don't set `dmPolicy` to `open`, or anyone who knows your bot's username can spend your balance.
- 🏠 **Don't touch `gateway.bind`**: keep it `loopback`; don't change it to LAN/public.
- 💰 **DeepSeek is pay-as-you-go**: keep an eye on your balance.
- 🔄 **Restart after config changes**: `openclaw gateway restart`.
- 🖥️ **Closing the console ≠ stopping the bot** (see checklist item 3).
- 😴 **Sleep/sign-out stops the bot**: disable system sleep if you want 24×7 uptime.
- 📁 **Don't put sensitive files in `~\.openclaw\workspace\`**: the agent can read them.
- ✅ **The first UAC prompt is a Node.js upgrade**: normal, click "Yes".

---

## 5. Troubleshooting quick reference

### "⚠️ Couldn't sign in to the AI service"
The key never reached the background process (usually because it was only set as an environment variable). Fix:

```powershell
openclaw models auth paste-api-key --provider deepseek
openclaw gateway restart
```

### "Config invalid: Unrecognized key: suppressNarrationOnMessaging"
That key does not exist — remove it. Use `reasoningDefault off` to hide thinking.

### The bot receives messages but doesn't reply
- Run `openclaw pairing list` to check for a pending pairing code.
- Run `openclaw config get channels.telegram` and confirm `allowlist` + your ID is in `allowFrom`.

### Status shows Telegram but messages don't flow (mainland China)
```powershell
curl.exe -s -o NUL -w "%{http_code}" https://api.telegram.org
```
If it fails, your proxy isn't applied system-wide; route the whole machine through a proxy, then `openclaw gateway restart`.

---

## 6. Security essentials

- Store keys in the credential store (`paste-api-key`), not in plaintext `openclaw.json`.
- Bind the gateway to `loopback` (127.0.0.1) only — never expose it.
- Telegram allowlist: only your numeric ID can command the bot.
- Backup = copy the whole `~\.openclaw\` folder (it contains keys — treat it as secret).

---

> For the full deployment process and detailed troubleshooting, see the `docs/` series in this repo.
