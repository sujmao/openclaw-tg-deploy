# OpenClaw + DeepSeek + Telegram 部署全流程（Windows）

> Deploy a personal OpenClaw gateway + DeepSeek model + Telegram bot on Windows, with Claude Code as the operator. A beginner-friendly, security-first runbook. **要求 Telegram 只顯示給人看的最終輸出，不顯示任何思考或工具調用。**

這份倉庫記錄了在 **Windows 10/11 + Claude Code + DeepSeek** 上，從零部署一個 **OpenClaw 個人助手 + Telegram 機器人** 的完整、可遷移流程。它面向**新手**，把「你（人）必須親手做的步驟」和「可以直接複製貼上的命令」明確分開，並把部署過程中踩過的坑（尤其是 DeepSeek 密鑰的坑）沉澱成可直接查閱的經驗。

---

## 這是什麼

部署完成後你會得到：

- 一台跑在你**自己電腦上**的 OpenClaw Gateway（綁定本機回環，不對外網暴露）。
- 一個掛在 **DeepSeek** 模型上的個人 AI 助手。
- 一個 **Telegram 機器人**，你可以隨時用手機私訊它，它回覆**只給人看的最終答案**（不顯示思考過程、不顯示工具調用）。
- 嚴格的**白名單**：只有你的 Telegram 帳號能指揮這個機器人。

```
你的 Windows 電腦
 ┌────────────────────────────────────────────┐
 │  OpenClaw Gateway  (127.0.0.1:18789, loopback) │
 │      └─ 模型: DeepSeek  ──► api.deepseek.com    │
 │      └─ 渠道: Telegram  ──► api.telegram.org    │
 └────────────────────────────────────────────┘
          ▲                    ▲
       你的手機             DeepSeek / Telegram 雲端
```

## 特性

- ✅ 全流程在**用戶空間**安裝，不寫系統目錄、不加開機服務（可選）、不改註冊表敏感項。
- ✅ 密鑰存入 OpenClaw **憑證庫**，不寫進 `openclaw.json` 明文。
- ✅ Telegram **白名單** + 群組預設關閉，別人無法操控你的機器人。
- ✅ `reasoningDefault: off`：Telegram 裡**看不到思考/工具調用**，只有最終輸出。
- ✅ 面向新手的**人工步驟**與**命令**分離；每步都有「如何驗證」。

## 前置條件（一句話版）

| 需要 | 說明 |
|---|---|
| Windows 10/11 | 家用/專業版均可 |
| DeepSeek 帳號 + API key | 到 platform.deepseek.com 創建 |
| Telegram 帳號 | 手機/桌面均可，用於建 bot 與驗收 |
| 能連到 api.deepseek.com | 中國大陸可直連 |
| 能連到 api.telegram.org | 中國大陸**需系統級代理**（見 docs/01） |
| Claude Code CLI | 用於執行命令（也可純手工） |

## 快速開始（TL;DR）

```powershell
# 1. 安裝（首次會彈 UAC 是 Node.js 升級，點「是」）
iwr -useb https://openclaw.ai/install.ps1 | iex

# 2. 配置 DeepSeek（貼上你的 key，回車）
openclaw models auth paste-api-key --provider deepseek
openclaw config set agents.defaults.model.primary deepseek/deepseek-v4-pro
openclaw config set agents.defaults.reasoningDefault off

# 3. 安全：網關只綁本機回環
openclaw config set gateway.bind loopback

# 4. 連接 Telegram（用 BotFather 拿到的 token 替換 <TOKEN>）
openclaw channels add --channel telegram --token <TOKEN>

# 5. 私訊你的 bot，然後批准你自己的帳號（用 bot 顯示的配對碼替換 <CODE>）
openclaw pairing approve --channel telegram <CODE> --notify

# 6. 白名單：只允許你的數字 ID（用你的 ID 替換 <ID>）
openclaw config set channels.telegram.dmPolicy allowlist
openclaw config set channels.telegram.allowFrom '["<ID>"]'

# 7. 驗證
openclaw config validate
openclaw status
```

> ⚠️ 上面每一行都有詳細說明、踩坑與驗收方法，**請照著 docs/ 順序讀**，不要只跑 TL;DR。

## 文檔索引

| 文檔 | 內容 |
|---|---|
| [docs/01-prerequisites.md](docs/01-prerequisites.md) | 帳號、網絡/代理、你要親手準備的東西 |
| [docs/02-install-openclaw.md](docs/02-install-openclaw.md) | 安裝 OpenClaw（含安全審計選項、UAC 解釋） |
| [docs/03-configure-deepseek.md](docs/03-configure-deepseek.md) | 配置 DeepSeek 模型 + 密鑰的坑 |
| [docs/04-connect-telegram.md](docs/04-connect-telegram.md) | 建 bot、連線、白名單 |
| [docs/05-security-hardening.md](docs/05-security-hardening.md) | 安全加固、隱藏思考/工具調用 |
| [docs/06-daily-operations.md](docs/06-daily-operations.md) | 重啟、手動開/關、24×7、日誌 |
| [docs/07-faq.md](docs/07-faq.md) | 常見問題（含「Couldn't sign in」） |
| [CLAUDE.md](CLAUDE.md) | 給 Claude Code 的操作指南 |

## 驗收清單（部署完成的定義）

裝好 ≠ 驗收通過。請在 Telegram 私訊 bot 一個**需要思考/推理**的問題，例如：

> 用中文解釋為什麼天空是藍色的，並給三個關鍵字

然後確認兩點：

1. ✅ bot 能**正常回覆**（不再是「Couldn't sign in to the AI service」）。
2. ✅ 訊息裡**只看得到最終答案**——看不到任何思考過程、推理步驟或工具調用痕跡。

兩點都滿足，即為部署完成。本倉庫的配置已實測通過：上述問題 bot 只回覆瑞利散射的解釋與三個關鍵字，無任何思考/工具輸出。

## 安全基線

- 密鑰**永不**寫入 `openclaw.json`、永不提交進 git（本倉庫已 `.gitignore` 所有常見密鑰檔名）。
- Gateway 只綁 `loopback`，不對局域網/公網開放。
- Telegram 白名單：僅你的數字 ID 可私訊。
- 群組預設 `allowlist`（即關閉）。

## 免責聲明

- 文中 `<TOKEN>`、`<ID>`、`sk-...` 均為佔位符，請替換成你自己的值，**不要把真實密鑰貼進公開倉庫或聊天**。
- DeepSeek 為按量付費，模型名稱與可用性以你帳戶實際為準。
- 本流程在 OpenClaw 2026.10.8 / Node 26.13.0 / Windows 10 19045 上實測通過；版本更新後個別命令可能變化，以 `openclaw <命令> --help` 為準。

## License

[MIT](LICENSE)
