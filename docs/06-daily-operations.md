# 06 · 日常運維：重啟、手動開關、24×7

部署成功後，這章是你**每天會用到**的操作手冊。

## 6.1 重啟電腦之後會發生什麼

- gateway 是以 **Windows 登入項（Scheduled Task）** 形式安裝的：**你登入 Windows 後它會自動啟動**。
- 因此**重啟電腦後**的標準流程是：

```powershell
# 1. 登入 Windows，等 1–2 分鐘讓 gateway 起來
# 2. 驗證
openclaw status
```

- 看到 Gateway `running`、Channels 表 Telegram `ON / OK`，就正常了。
- 若 gateway 沒起來（極少見），手動拉一把：

```powershell
openclaw daemon start
```

## 6.2 手動啟動 / 關閉環境

| 你要做的事 | 命令 | 說明 |
|---|---|---|
| **開**（啟動 bot） | `openclaw daemon start` | 啟動 gateway 服務；登入時通常已自動 |
| **關**（停 bot） | `openclaw daemon stop` | 停止 gateway，bot 不再回應 |
| **重啟**（改了配置後） | `openclaw gateway restart` | 讓配置/憑證立即生效 |
| **查狀態** | `openclaw status` | 一屏看清 gateway/渠道/會話 |
| **看即時日誌** | `openclaw logs --follow` | 排障必備，Ctrl+C 退出 |
| **測模型是否通** | `openclaw agent --message "hi"` | 端到端走一遍 gateway+模型 |

> 典型「改了配置」場景：改了白名單、換了模型、更新了密鑰——先 `openclaw gateway restart` 再驗證。

## 6.3 關於「24 小時不關機」

- **登入項的性質**：gateway 需要一個**已登入的桌面會話**。你登入後它就常駐；若登出/註銷 Windows，或電腦睡眠/休眠，gateway 會停。
- 若你要的是**真正的「未登入也常駐」開機服務**（例如一台永不登出的伺服器），需要把 gateway 裝成系統服務並配置好憑據，這超出了本流程「家用、安全、可逆」的默認範圍。有需要時再單獨評估。
- 桌面電腦想讓它 24×7 穩定，實務建議：關閉「睡眠/休眠」，讓電腦只關顯示器、不睡眠（電源設定裡把睡眠改為「永不」）。

## 6.4 更新 OpenClaw

```powershell
openclaw update       # 檢查並更新
openclaw doctor       # 更新後的健康檢查
```

更新後若出現「Plugin deepseek 升級未完成」之類的警告，按提示 `openclaw update repair` 或 `openclaw doctor --fix` 修復。

## 6.5 備份

你的重要狀態都集中在：

```
C:\Users\<你>\.openclaw\
├── openclaw.json        # 配置
├── state\openclaw.sqlite # 憑據 + 會話
└── agents\              # agent 記憶/工作區
```

- 定期備份整個 `.openclaw` 目錄即可。
- 密鑰在 `state\openclaw.sqlite` 裡，備份文件同樣要當機密對待。

## 6.6 忘了密鑰 / token 放哪了

- DeepSeek key：登入 platform.deepseek.com 重新創建（舊的建議作廢）。
- Telegram bot token：找 BotFather 發 `/mybots` → 選你的 bot → 可重新生成 token。
- 重新設置密鑰：重跑 `openclaw models auth paste-api-key --provider deepseek`，然後 `openclaw gateway restart`。

進入 [`07-faq.md`](07-faq.md) 看常見坑。

## 6.7 快速開啟 / 關閉控制檯（Control UI）

| 動作 | 命令 / 做法 |
|---|---|
| 開啟（網頁控制台） | `openclaw dashboard` —— 自動帶 token 在瀏覽器打開 `http://127.0.0.1:18789/` |
| 開啟（終端 UI） | `openclaw tui`（或 `openclaw terminal`） |
| 關閉（網頁） | 關掉瀏覽器分頁即可 |
| 關閉（終端） | 在 TUI 內按 `Ctrl+C` 退出 |

> ⚠️ **關閉控制檯 ≠ 關閉 bot**：bot 由後台 gateway 常駐，關掉控制台它照樣在 Telegram 回應；真要停 bot 用 `openclaw daemon stop`。
> 所有 `openclaw` 命令在任何目錄都能直接跑，不需要先 `cd`（詳見 [`08-paths-and-workspace.md`](08-paths-and-workspace.md)）。
