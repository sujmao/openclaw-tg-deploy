# 01 · 準備工作（你要親手準備的東西）

開始之前，請先把下面幾樣東西備齊。這些是**只能由你本人完成**的步驟，Claude Code 無法代勞。

## 1. 一個 DeepSeek API key

1. 打開 <https://platform.deepseek.com>，註冊 / 登入。
2. 在「API Keys」頁面點「創建 API key」，起個名字（例如 `openclaw-tg`）。
3. **立刻複製**那一串 `sk-...` 開頭的密鑰並妥善保存（它只顯示一次）。

> - DeepSeek 是**按量付費**，新帳號通常要先充值少量餘額才能調用。
> - 密鑰等同於「可以花你帳戶餘額的鑰匙」，**不要**貼到公開倉庫、群聊或截圖。
> - 這份倉庫全程用 `<你的DeepSeek密鑰>` 佔位，不要提交真實值。

## 2. 一個 Telegram 帳號 + 一個 bot

1. 手機或桌面安裝 Telegram，註冊並登入。
2. 在 Telegram 裡搜尋 **@BotFather**（認證過的官方帳號）。
3. 給它發 `/newbot`，按提示：
   - 先給 bot 起一個**顯示名**（可中文，如「我的助手」）；
   - 再起一個**用戶名**（必須以 `bot` 結尾，如 `my_assistant_2026_bot`）。
4. BotFather 會回覆一串 **bot token**（形如 `1234567890:AAE...`），**立刻保存**。

> 這串 token 就是機器人的登入憑證，等同於機器人的密碼，同樣不要外洩。

## 3. 知道自己的 Telegram 數字 ID

白名單需要你本人的數字 ID（一長串數字，不是用戶名）。兩種取得方式：

- **方式 A（推薦，配合配對流程）**：後續部署時，你私訊自己的 bot，OpenClaw 會在批准時顯示你的數字 ID（見 `docs/04`）。
- **方式 B**：在 Telegram 搜尋 **@userinfobot**，給它發任意訊息，它會回覆你的數字 ID。

## 4. 網絡連通性（重要）

| 目標 | 中國大陸 | 說明 |
|---|---|---|
| `api.deepseek.com` | ✅ 可直連 | DeepSeek 是國內服務 |
| `api.telegram.org` | ⚠️ 需代理 | Telegram 官方 API 在大陸無法直連 |

**關鍵結論**：機器人跑在**你自己的電腦**上，所以你的電腦必須能連到 Telegram 的 API。如果你的電腦有系統級代理（如 TUN/全局模式的 Clash、V2Ray 等），OpenClaw 進程通常會繼承系統代理而自動走通；若只有瀏覽器代理，還需確保命令行進程也能用（例如設置 `HTTP_PROXY`/`HTTPS_PROXY` 環境變數，或用 TUN 模式讓系統層面全局生效）。

> 驗證方法：在 PowerShell 裡跑
> ```powershell
> curl.exe -s -o NUL -w "%{http_code}" https://api.telegram.org
> ```
> 返回 `200`/`404` 等正常 HTTP 狀態碼即說明能連到；超時/失敗則代理未生效。
> 這一步不通，後面 bot 會「有狀態但收發不了訊息」。

## 5. Windows 與 Node.js

- Windows 10/11 均可。OpenClaw 安裝器會自動處理 Node.js：若版本過舊，會觸發一次 **UAC 提權彈窗**（Node.js MSI 升級），點「是」即可（詳見 `docs/02`）。
- 全程安裝在**用戶空間**，不寫系統目錄、不裝惡意開機項，安全基線見 `docs/05`。

## 準備完成檢查清單

- [ ] 已拿到 DeepSeek API key（`sk-...`）
- [ ] 已用 BotFather 建好 bot，拿到 bot token
- [ ] 已知自己的 Telegram 數字 ID（或知道配對流程能拿到）
- [ ] 電腦能連到 `api.telegram.org`（`curl` 驗證過）
- [ ] 準備好使用 PowerShell（開始選單搜「PowerShell」）

備齊後，進入 [`02-install-openclaw.md`](02-install-openclaw.md)。
