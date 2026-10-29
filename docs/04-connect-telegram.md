# 04 · 連接 Telegram 機器人

## 4.1 把 bot token 交給 OpenClaw

用你在 `docs/01` 從 BotFather 拿到的 token 替換 `<TOKEN>`：

```powershell
openclaw channels add --channel telegram --token <TOKEN>
```

- token 形如 `1234567890:AAE...`，整體是一串，直接整段貼上。
- 成功後，OpenClaw 會開始輪詢 Telegram。

## 4.2 批准你自己的帳號（配對）

默認私訊策略是「需要配對」。做法：

1. 用手機打開 Telegram，搜到你剛建的 bot，給它**發任意一條訊息**（如 `hi`）。
2. bot 會回覆一個**配對碼**（一串字母數字，如 `UWQ595RJ`）。
3. 回到 PowerShell，用這個碼批准（把 `<CODE>` 換成實際配對碼）：

```powershell
openclaw pairing approve --channel telegram <CODE> --notify
```

- 這一步會同時把你的**數字 ID** 記錄為「已批准發送者」。批准後可用 `openclaw pairing list` 查看。
- 記下這個數字 ID，下一步白名單會用到。

## 4.3 白名單加固：只允許你一個人

```powershell
openclaw config set channels.telegram.dmPolicy allowlist
openclaw config set channels.telegram.allowFrom '["<ID>"]'
openclaw config set channels.telegram.groupPolicy allowlist
```

把 `<ID>` 換成你的**數字 ID**（純數字，不加 `@`）。例如你的 ID 是 `123456789`，就寫 `'["123456789"]'`。

含義：

- `dmPolicy: allowlist` + `allowFrom`：**只有**這個 ID 能私訊指揮你的 bot，其他任何人發訊息都被忽略。
- `groupPolicy: allowlist`：群組默認不響應，防止別人把你拉進群誘導 bot 幹活。

> 設置時若命令行先後順序導致一條臨時警告「allowFrom is empty; all DMs will be dropped」，不用慌——只要最後 `allowFrom` 已填上、`config validate` 通過即可。

## 4.4 驗證

```powershell
openclaw config validate    # 應顯示 Config valid
openclaw status             # Channels 表裡 Telegram 應為 ON / OK
```

然後回到 Telegram，私訊 bot 一個問題，應該能收到回覆了。

## 最終配置示例（已打碼）

```json5
{
  channels: {
    telegram: {
      enabled: true,
      botToken: "<TOKEN 佔位>",
      dmPolicy: "allowlist",
      allowFrom: [ "<你的數字ID>" ],
      groupPolicy: "allowlist"
    }
  },
  commands: { ownerAllowFrom: [ "telegram:<你的數字ID>" ] }
}
```

進入 [`05-security-hardening.md`](05-security-hardening.md)。
