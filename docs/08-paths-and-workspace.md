# 08 · 命令在哪跑、工作空間位置與限制

> 給「只會用 Claude Code」的新手：OpenClaw 跟 Claude Code 的運行方式**很不一樣**，這一頁專門講清楚。

## 8.1 `openclaw` 命令在哪裡執行？（答案：任何地方）

- `openclaw` 是**全局命令**，安裝時已放進系統 PATH（實際文件在 `C:\Users\<你>\AppData\Roaming\npm\openclaw.ps1`）。
- 所以**不需要 `cd` 到任何資料夾**：打開任意一個 PowerShell 窗口，直接打 `openclaw status`、`openclaw dashboard`、`openclaw daemon stop` 都能用。
- 對比：Claude Code 通常要「進到某個專案資料夾」再跑 `claude`（它有專案範圍）；而 `openclaw` 操作的是全局的 `~\.openclaw` 狀態，跟當前目錄無關。
- 真正的「常駐」是後台 gateway（Windows 登入項），它**不依賴你開著的終端**；你打的 `openclaw ...` 命令只是「遙控器」。

## 8.2 OpenClaw 的資料放在哪裡

所有東西都在你的用戶主目錄下 `C:\Users\<你>\.openclaw\`（縮寫 `~\.openclaw`）：

| 項目 | 位置 |
|---|---|
| 配置文件 | `~\.openclaw\openclaw.json` |
| 密鑰 / 憑據 / 狀態 | `~\.openclaw\state\openclaw.sqlite` |
| **agent 工作區**（讀寫文件的地方） | `~\.openclaw\workspace\` |
| agent 記憶 / 會話資料庫 | `~\.openclaw\agents\main\agent\openclaw-agent.sqlite` |
| 日誌 | `~\.openclaw\logs\` |
| 命令本體 | `C:\Users\<你>\AppData\Roaming\npm\` |

> 注意「工作空間」有兩層意思：
> 1. **OpenClaw 程序自己的數據目錄** = `~\.openclaw\`；
> 2. **agent 能讀寫文件的「工作區」** = `~\.openclaw\workspace\`（默認只在這裡活動）。

## 8.3 如何「確定」agent 的工作區

```powershell
openclaw config get agents.defaults.workspace
```

- 若回「unset / runtime default applies」，表示用的是默認值：`~\.openclaw\workspace\`。
- 直接看 `~\.openclaw\workspace\` 目錄內容（裡面有 `AGENTS.md`、`BOOTSTRAP.md`、`SOUL.md`、`USER.md` 等），那就是 agent 的「家」。

## 8.4 如何「限制」agent 的工作區（安全）

1. **默認已夠安全**：agent 只在 `~\.openclaw\workspace\` 內讀寫，不碰你其他磁碟的文件；「Host desktop（桌面控制）」默認是關閉的。
2. **換一個工作區 / 限制到指定資料夾**：

   ```powershell
   openclaw config set agents.defaults.workspace "D:\某個資料夾"
   openclaw gateway restart
   ```

3. **更強隔離（可選）**：`openclaw sandbox` 可用 Docker 把 agent 關進容器（家用場景默認不需要）。
4. **反過來也要注意**：別把敏感文件放進 `~\.openclaw\workspace\`，因為 agent 讀得到那裡面。
