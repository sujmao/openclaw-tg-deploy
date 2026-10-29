# CLAUDE.md — 給 Claude Code 的操作指南

本文件告訴 Claude Code 如何「讀取並執行」這套 OpenClaw + DeepSeek + Telegram 部署流程。
當用戶說「照這個 repo 部署」「幫我裝」「重跑流程」時，按本文件行事。

## 角色與目標

你是部署執行者。目標：在**用戶自己的 Windows 電腦**上，把 OpenClaw Gateway + DeepSeek 模型 + Telegram 機器人裝起來並驗收通過，硬性要求是 **Telegram 只顯示給人看的最終輸出，不顯示思考或工具調用**。

## 必須先讀的文檔（按順序）

1. `docs/01-prerequisites.md` — 先確認用戶的帳號/網絡/密鑰是否備齊，缺哪樣先停下要哪樣。
2. `docs/02-install-openclaw.md`
3. `docs/03-configure-deepseek.md`
4. `docs/04-connect-telegram.md`
5. `docs/05-security-hardening.md`
6. `docs/06-daily-operations.md`（含重啟與手動開關）

`docs/07-faq.md` 是排障庫，遇到報錯先查它。

## 核心原則（鐵律）

1. **密鑰不落盤、不進倉庫、不進 `openclaw.json`。**
   - DeepSeek key、Telegram bot token、用戶數字 ID 一律用佔位符，真實值只在用戶本地命令中輸入。
   - 任何「回顯/讀取密鑰內容」的操作都禁止；設置憑證用管道餵入、不打印。
2. **密鑰用憑證庫，不用環境變數。**（這是本流程最重要的坑，見 FAQ-1）
   - 用 `openclaw models auth paste-api-key --provider deepseek` 存進 auth profile。
   - 不要只靠 `$env:DEEPSEEK_API_KEY`：常駐 gateway 進程不會自動刷新新加的環境變數。
3. **Gateway 只綁 loopback**：`gateway.bind: loopback`，不要改成 lan/公網。
4. **Telegram 白名單**：`dmPolicy: allowlist` + `allowFrom: ["<用戶數字ID>"]`，群組關閉。
5. **隱藏思考**：`agents.defaults.reasoningDefault: off`。
   - 注意：`suppressNarrationOnMessaging` 在 2026.10.8 **不存在**，不要使用。

## 標準執行序列（可複製的原子步驟）

```powershell
# 安裝
iwr -useb https://openclaw.ai/install.ps1 | iex          # 首次 UAC 是 Node.js 升級
openclaw --version                                        # 驗證安裝

# DeepSeek
openclaw models auth paste-api-key --provider deepseek    # 讓用戶貼 key
openclaw config set agents.defaults.model.primary deepseek/deepseek-v4-pro
openclaw config set agents.defaults.reasoningDefault off
openclaw config set gateway.bind loopback

# Telegram
openclaw channels add --channel telegram --token <TOKEN>  # 用戶提供 token
openclaw pairing approve --channel telegram <CODE> --notify   # 用戶先私訊 bot 拿配對碼
openclaw config set channels.telegram.dmPolicy allowlist
openclaw config set channels.telegram.allowFrom '["<ID>"]'
openclaw config set channels.telegram.groupPolicy allowlist

# 驗證
openclaw config validate
openclaw status
```

## 每個階段的驗收門（gate）

- **安裝後**：`openclaw --version` 返回版本號。
- **模型後**：`openclaw agent --message "Reply with exactly: OK" --thinking off` 返回 `OK`。
- **渠道後**：`openclaw status` 的 Channels 表裡 Telegram = `ON / OK`。
- **白名單後**：`openclaw config get channels.telegram` 顯示 `allowlist` + 正確 `allowFrom`。
- **最終驗收（需用戶）**：用戶在 Telegram 私訊 bot 一個需要思考的問題，確認①正常回覆、②看不到思考/工具調用。兩點都過才算完成。

## 需要「人」親手做的步驟（你不能代勞）

- 註冊 DeepSeek、充值、創建並複製 API key。
- 安裝 Telegram 並在 BotFather 建 bot、拿 token。
- 取得自己的 Telegram 數字 ID（@userinfobot，或配對時自動得知）。
- Windows 彈出的 UAC 提示（Node.js 升級）——由用戶點「是」。
- 重啟電腦後登入 Windows。

這些步驟的詳細說明在 `docs/01` 與 `docs/04`，向用戶要對應值時，引用具體文檔段落。

## 重啟與手動開關（常用運維命令）

| 動作 | 命令 |
|---|---|
| 手動啟動 | `openclaw daemon start` |
| 手動停止 | `openclaw daemon stop` |
| 重啟（改配置後） | `openclaw gateway restart` |
| 查狀態 | `openclaw status` |
| 看日誌 | `openclaw logs --follow` |

- gateway 是**登入項**（Scheduled Task）：用戶登入 Windows 後自動啟動，重啟電腦後等 1–2 分鐘再驗證。
- 若需要「未登入也常駐」的真·開機服務，見 `docs/06`。

## 排障速查

- 「Couldn't sign in to the AI service」→ 密鑰沒進 gateway 進程，見 FAQ-1：用 `paste-api-key` + `openclaw gateway restart`。
- 「Config invalid: Unrecognized key: suppressNarrationOnMessaging」→ 刪掉該 key，見 FAQ-2。
- Telegram 沒反應 → 檢查白名單/配對/代理，見 FAQ-4、FAQ-5。

## 發佈規範（Release）

每次發佈「部署後參考手冊」，必須**同時出三個語言版本**，文件名固定：

- `manual/POST-DEPLOY-MANUAL.zh-TW.md`（繁體中文）
- `manual/POST-DEPLOY-MANUAL.zh-CN.md`（简体中文）
- `manual/POST-DEPLOY-MANUAL.en.md`（English）

三版內容同步、同一版本號，用 `gh release create vX.Y.Z` 把三個文件都作為附件上傳。改動任何一版時，必須同步翻譯並更新另外兩版。
