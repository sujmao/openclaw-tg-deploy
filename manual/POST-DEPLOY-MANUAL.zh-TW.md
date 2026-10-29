# OpenClaw + Telegram 部署後參考手冊（給人看的）

> 版本 v1.0.0 · 適用 OpenClaw 2026.10.8 · Windows 10/11 · DeepSeek
> 這本手冊是「裝好之後」給**人類**日常翻閱用的。所有命令都能在任意 PowerShell 窗口直接執行，**不需要先 `cd` 到哪個資料夾**。

---

## 0. 先記住三句話

1. **命令不用 cd**：`openclaw` 是全局命令，任何位置都能跑（跟 Claude Code 要先進專案資料夾不同）。
2. **東西都在 `C:\Users\<你>\.openclaw\`**：配置、密鑰、agent 工作區、記憶全在這個資料夾。
3. **關控制檯 ≠ 關 bot**：關掉瀏覽器/終端，bot 照樣在後台回應；真要停，用 `openclaw daemon stop`。

---

## 1. 位置速查表

| 要找什麼 | 位置 |
|---|---|
| 配置文件 | `C:\Users\<你>\.openclaw\openclaw.json` |
| 密鑰 / 憑據 / 狀態 | `C:\Users\<你>\.openclaw\state\openclaw.sqlite` |
| agent 工作區（它讀寫文件的地方） | `C:\Users\<你>\.openclaw\workspace\` |
| agent 記憶 / 會話 | `C:\Users\<你>\.openclaw\agents\main\agent\openclaw-agent.sqlite` |
| 日誌 | `C:\Users\<你>\.openclaw\logs\` |

> 把 `<你>` 換成你的 Windows 用戶名。最快的辦法：在 PowerShell 打 `echo $env:USERPROFILE`。

### 確定 / 限制 agent 工作區

- **確定**：`openclaw config get agents.defaults.workspace`——若回「unset，runtime default applies」，默認就是 `~\.openclaw\workspace\`。
- **限制**：默認 agent 只在 `~\.openclaw\workspace\` 內活動、不碰你其他磁碟文件。要換位置：`openclaw config set agents.defaults.workspace "D:\某資料夾"` + `openclaw gateway restart`。**別把敏感文件放進 `workspace\`**，agent 讀得到。

---

## 2. 日常操作速查

| 你想做的事 | 命令 / 做法 |
|---|---|
| 查狀態（最重要，先跑這個） | `openclaw status` |
| **開** bot | `openclaw daemon start` |
| **關** bot | `openclaw daemon stop` |
| 改了配置後重啟 | `openclaw gateway restart` |
| 看即時日誌（排障） | `openclaw logs --follow`（Ctrl+C 退出） |
| 測模型通不通 | `openclaw agent --message "hi"` |
| 打開控制檯（網頁） | `openclaw dashboard` |
| 打開控制檯（終端） | `openclaw tui` |
| 關閉控制檯 | 關瀏覽器分頁 / TUI 內 Ctrl+C |

> **重啟電腦後**：登入 Windows，等 1–2 分鐘，`openclaw status` 看到 Gateway `running`、Telegram `ON / OK` 即正常；沒起來就 `openclaw daemon start`。

---

## 3. 驗收清單（怎麼算「裝好了」）

在 Telegram 私訊 bot 一個需要思考的問題（例：`用中文解釋為什麼天空是藍色的，並給三個關鍵字`），確認：

1. ✅ 能**正常回覆**（不是「Couldn't sign in…」）。
2. ✅ **只看得到最終答案**，看不到思考/推理/工具調用。
3. ✅ 能**快速開/關控制檯**（`openclaw dashboard` → 關分頁）。

---

## 4. 新手注意事項

- 🔑 **密鑰/token 絕不外洩**：DeepSeek key、bot token 等同密碼，勿貼公開倉庫/群聊/截圖。
- 👥 **白名單只加自己**：勿把 `dmPolicy` 改成 `open`，否則知道 bot 用戶名的人都能用你的餘額。
- 🏠 **別動 `gateway.bind`**：保持 `loopback`，勿改 lan/公網。
- 💰 **DeepSeek 按量付費**：留意餘額。
- 🔄 **改配置後要重啟**：`openclaw gateway restart` 才生效。
- 🖥️ **關控制檯 ≠ 停 bot**（見驗收清單第 3 條）。
- 😴 **睡眠/登出會停 bot**：想 24×7 常駐，關閉系統睡眠。
- 📁 **別把敏感文件放進 `~\.openclaw\workspace\`**：agent 讀得到那裡面。
- ✅ **首次 UAC 彈窗是 Node.js 升級**：正常，點「是」。

---

## 5. 常見問題速查

### 「⚠️ Couldn't sign in to the AI service」
密鑰沒進後台進程（通常是只設了環境變數）。修法：

```powershell
openclaw models auth paste-api-key --provider deepseek
openclaw gateway restart
```

### 「Config invalid: Unrecognized key: suppressNarrationOnMessaging」
這個鍵不存在，刪掉即可。隱藏思考用 `reasoningDefault off`。

### bot 收到訊息但不回應
- 先 `openclaw pairing list` 看有沒有待批准的配對碼。
- 再 `openclaw config get channels.telegram` 確認 `allowlist` + 你的 ID 在 `allowFrom` 裡。

### status 有 Telegram 但收發不了（中國大陸）
```powershell
curl.exe -s -o NUL -w "%{http_code}" https://api.telegram.org
```
不通就是代理沒生效，讓整台電腦走代理後 `openclaw gateway restart`。

---

## 6. 安全要點

- 密鑰用憑證庫（`paste-api-key`），不落 `openclaw.json` 明文。
- gateway 只綁 `loopback`（127.0.0.1），不暴露。
- Telegram 白名單：只有你的數字 ID 能指揮 bot。
- 備份 = 複製整個 `~\.openclaw\`（備份含密鑰，當機密對待）。

---

> 完整部署流程與排障詳解，見同倉庫的 `docs/` 系列文檔。
