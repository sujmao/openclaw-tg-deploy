# 07 · 常見問題（排障庫）

## FAQ-1：Telegram 回覆「⚠️ Couldn't sign in to the AI service」

**現象**：bot 能收到訊息，但回覆一句「Couldn't sign in to the AI service. Sign in again under Models in the Control UI or run openclaw configure.」

**根因**：gateway 後台進程裡沒有 DeepSeek 密鑰。最常見的誘因是——密鑰是通過 `$env:DEEPSEEK_API_KEY` 設置的，而 gateway 進程**早於**該設置就已啟動；Windows 進程不會自動刷新之後新增的環境變數。

**修法**（本流程已採用）：

```powershell
openclaw models auth paste-api-key --provider deepseek   # 存進憑證庫
openclaw gateway restart                                 # 重啟 gateway 生效
openclaw agent --message "Reply with exactly: OK" --thinking off   # 驗證
```

返回 `OK` 即解決。

**為什麼這比環境變數好**：憑證庫由 gateway 直接讀取，不依賴進程啟動時有沒有繼承到環境變數。

## FAQ-2：`config validate` 報「Unrecognized key: suppressNarrationOnMessaging」

**現象**：配置裡寫了 `suppressNarrationOnMessaging`，驗證報錯「Config invalid」。

**原因**：這個鍵在 OpenClaw 2026.10.8 裡**不存在**（它來自未來版本/未合併的 PR）。隱藏思考請用 `reasoningDefault: off`，工具調用在 Telegram 私訊默認本就不顯示。

**修法**：把 `suppressNarrationOnMessaging` 從配置中刪掉即可。

## FAQ-3：`openclaw` 不是可識別的命令

**現象**：安裝成功但命令找不到。

**修法**：關掉當前 PowerShell，重新開一個窗口（安裝器更新了用戶 PATH，需新進程才生效）。若仍不行，重跑一次安裝。

## FAQ-4：bot 收到訊息但不回應 / 顯示 pending

- 若是「待批准/配對」：你還沒批准自己，跑 `openclaw pairing list` 看有沒有待批准的配對碼，再 `openclaw pairing approve --channel telegram <CODE> --notify`。
- 若已批准仍不回應：確認 `dmPolicy` 是 `allowlist` 且 `allowFrom` 裡有你的數字 ID；`openclaw config get channels.telegram` 核對。
- 確認 `openclaw status` 裡 Telegram 是 `ON / OK`。

## FAQ-5：status 顯示 Telegram 有狀態但收發不了訊息（中國大陸）

**現象**：`openclaw status` 裡 Telegram 看起來連上了，但實際收發不到。

**原因**：`api.telegram.org` 在大陸無法直連，bot 的長輪詢實際沒通。

**排查**：

```powershell
curl.exe -s -o NUL -w "%{http_code}" https://api.telegram.org
```

- 通：返回正常 HTTP 狀態碼。
- 不通/超時：需要讓整台電腦走代理（TUN/全局模式），或給進程設 `HTTP_PROXY`/`HTTPS_PROXY`。確認後 `openclaw gateway restart` 再試。

## FAQ-6：模型名報錯 / 模型不可用

- 先 `openclaw models list` 看你帳戶實際可用模型。
- DeepSeek 模型名以 `deepseek/` 為前綴，例如 `deepseek/deepseek-v4-pro`、`deepseek/deepseek-flash`。
- 帳戶欠費/無餘額也會導致調用失敗，先充值。

## FAQ-7：如何徹底卸載重來

```powershell
openclaw daemon stop
openclaw daemon uninstall
# 視需要刪掉配置（會清掉密鑰與會話）：
# Remove-Item -Recurse -Force "$env:USERPROFILE\.openclaw"
```

---

> 其他問題先查官方：`openclaw doctor`、`openclaw status --deep`、<https://docs.openclaw.ai/faq>。
