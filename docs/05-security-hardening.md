# 05 · 安全加固

這一章是「傾向於安全」的你最該細讀的。核心思想：**最小暴露、最小信任、密鑰不落明文**。

## 5.1 網關只綁本機回環

```powershell
openclaw config set gateway.bind loopback
```

- `loopback`：只監聽 `127.0.0.1`，**只有你本機**能訪問控制面（Control UI 在 `http://127.0.0.1:18789`）。
- **不要**改成 `lan`（局域網）或 `tailnet`/公網，除非你明確需要遠程訪問且懂得自己在做什麼。
- 網關本身有**認證 token**（OpenClaw 自動生成），即便綁回環，控制面也要憑證才能操作。可用 `openclaw config get gateway.auth` 查看模式（token）。

## 5.2 密鑰不落明文 `openclaw.json`

| 做法 | 是否推薦 |
|---|---|
| `openclaw models auth paste-api-key --provider deepseek` | ✅ 存憑證庫 |
| 設 `$env:DEEPSEEK_API_KEY` 環境變數 | ⚠️ 可用但常駐進程不刷新，易踩坑 |
| 把 `apiKey` 直接寫進 `openclaw.json` | ❌ 明文落盤，且易被誤提交 |

- 部署完後可用 `Select-String -Path $env:USERPROFILE\.openclaw\openclaw.json -Pattern 'sk-'` 自查：應**找不到**任何明文 key。
- 本倉庫 `.gitignore` 已攔截常見密鑰檔名，雙保險。

## 5.3 Telegram 訪問控制（最重要）

- **私訊白名單**：`dmPolicy: allowlist` + `allowFrom: ["<你的ID>"]`。只有你本人能指揮 bot。
- **群組關閉**：`groupPolicy: allowlist`（默認不響應任何群）。
- **不要**用 `dmPolicy: open`（任何人可指揮，等於把 bot 控制權送人）。

> 想像一下後果：如果 `dmPolicy` 是 open，任何一個知道你 bot 用戶名的人，都能用你的 DeepSeek 餘額、以你的身份讓 agent 幹活（查文件、發訊息……）。所以白名單**不是可選項**。

## 5.4 隱藏思考與工具調用（硬性要求）

- `agents.defaults.reasoningDefault: off` → Telegram 不顯示思考過程。
- Telegram 私訊默認**不渲染**工具調用，只發最終文本。
- 驗收標準：私訊一個需要思考的問題，Telegram 裡**只看到給人看的最終答案**。

## 5.5 關於微信頻道（暫緩）

OpenClaw 官方微信插件 `@tencent-weixin/openclaw-weixin` 曾存在訪問控制漏洞（舊版白名單/掃碼回退可能「放行任意發送者」）。**本流程暫不接入微信**；若日後要接，務必：

1. 使用最新版插件；
2. 手動核對白名單確實生效（用非白名單帳號試發、確認被拒）；
3. 綁定後再審計一遍 `openclaw status --deep`。

> 為什麼微信比 Telegram 危險、別人怎麼「用聊天」遠程指揮你的 agent、以及「先後綁定多個微信號是否會放行歷史用戶」的完整解析，見 [`09-wechat-security.md`](09-wechat-security.md)。

## 5.6 定期做一次安全自檢

```powershell
openclaw security audit      # 快速安全審計
openclaw status --deep       # 深度檢查（含渠道狀態）
openclaw config get channels.telegram   # 確認白名單仍在
```

進入 [`06-daily-operations.md`](06-daily-operations.md)。
