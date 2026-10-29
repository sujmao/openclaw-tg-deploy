# 03 · 配置 DeepSeek 模型

這一步把 DeepSeek 設為助手的主模型，並把密鑰安全地存進 OpenClaw 的憑證庫。

## 3.1 保存密鑰到憑證庫（關鍵）

```powershell
openclaw models auth paste-api-key --provider deepseek
```

執行後它會提示你**粘貼密鑰**，粘貼 `sk-...` 後回車。成功會顯示類似：

```
Auth profile: deepseek:manual (deepseek/api_key)
```

### ⚠️ 為什麼用這個命令，而不是設環境變數？（最重要的坑）

很多教程會教你 `$env:DEEPSEEK_API_KEY = "sk-..."`。**在 Windows 上這會踩坑**：

- OpenClaw 的 gateway 是一個**常駐後台進程**。Windows 進程啟動後，**不會自動刷新**之後才新增的環境變數。
- 結果：命令列單獨測試時明明能通，但 bot 真收到訊息時，後台進程裡根本沒有 key，回覆你
  「⚠️ Couldn't sign in to the AI service.」

用 `paste-api-key` 把密鑰存進 OpenClaw **自己的憑證庫**（`~\.openclaw\state\openclaw.sqlite`，與 `openclaw.json` 分離），gateway 直接讀取，就不依賴「進程能不能繼承環境變數」這層不確定性。**這是本流程最值錢的一條經驗**。詳細排障見 [`07-faq.md` FAQ-1](07-faq.md)。

## 3.2 設置主模型

```powershell
openclaw config set agents.defaults.model.primary deepseek/deepseek-v4-pro
```

DeepSeek 帳戶常見可選模型（以你帳戶實際為準，可用 `openclaw models list` 查看）：

| 模型 | 定位 |
|---|---|
| `deepseek/deepseek-flash` | 更快更便宜，日常問答夠用 |
| `deepseek/deepseek-v4-pro` | 更強，適合較複雜任務（本流程預設） |

## 3.3 關閉思考顯示（硬性要求）

```powershell
openclaw config set agents.defaults.reasoningDefault off
```

- 這一步讓 Telegram 裡**只顯示最終答案，不顯示模型的思考過程**。
- 工具調用（tool calls）在 Telegram 私訊裡**預設就不會顯示**，只會把最終文本發給你，無需額外開關。
- ⚠️ 有一個看起來相關、但**不存在的**鍵 `suppressNarrationOnMessaging`——它在 OpenClaw 2026.10.8 裡無效，設置了會導致 `config validate` 報錯，**不要用**。

## 3.4 安全：網關只綁本機回環

```powershell
openclaw config set gateway.bind loopback
```

讓 gateway 只監聽 `127.0.0.1`，不對局域網/公網暴露（詳見 `docs/05`）。

## 3.5 驗證模型通了

```powershell
openclaw agent --message "Reply with exactly: OK" --thinking off
```

返回 `OK` 即表示 DeepSeek 已打通。

> 若這一步返回與「sign in / auth / api key」相關的錯誤，先看 `docs/07` FAQ-1。

## 本節完成後

配置大致為（已打碼示例，供你對照）：

```json5
{
  agents: { defaults: { model: { primary: "deepseek/deepseek-v4-pro" }, reasoningDefault: "off" } },
  gateway: { bind: "loopback" }
}
```

進入 [`04-connect-telegram.md`](04-connect-telegram.md)。
