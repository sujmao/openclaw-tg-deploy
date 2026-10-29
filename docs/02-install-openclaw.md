# 02 · 安裝 OpenClaw

## 官方安裝（推薦新手直接這樣做）

在 **PowerShell** 裡貼上並回車：

```powershell
iwr -useb https://openclaw.ai/install.ps1 | iex
```

- `iwr -useb`：下載腳本（忽略 IE 緩存）。
- `| iex`：執行下載到的腳本。

### 安裝過程中你可能會遇到

1. **UAC 彈窗（用戶帳戶控制）**：若你本機 Node.js 版本過舊，安裝器會觸發 Node.js 的 MSI 升級，彈出「是否允許此應用變更你的裝置」。這是**升級 Node.js 運行時**，點「是」即可。（實測：從 Node 24 升級到 26，原位升級，不破壞已裝的 npm 全局包。）
2. 等待一兩分鐘，出現成功提示即完成。

### 驗證安裝

```powershell
openclaw --version
```

看到類似 `OpenClaw 2026.x.x` 的版本號即成功。若提示「找不到 openclaw 命令」，關掉當前 PowerShell 重開一個（安裝器更新了 PATH，需新開窗口才生效）。

## 安全審計安裝（給「傾向安全」的你）

如果你不想直接執行網上下載的腳本，可以先下載審計、再執行：

```powershell
# 1. 下載到本地（不執行）
New-Item -ItemType Directory -Force "$env:TEMP\oc-audit"
Invoke-WebRequest -UseBasicParsing https://openclaw.ai/install.ps1 -OutFile "$env:TEMP\oc-audit\install.ps1"

# 2. 檢查腳本裡有沒有可疑的系統級操作（舉例，可自己擴充）
Select-String -Path "$env:TEMP\oc-audit\install.ps1" -Pattern 'HKLM|HKCU|system32|schtasks|New-Service|Set-ItemProperty' -SimpleMatch

# 3. 確認無可疑後，本地執行（-NoOnboard 表示跳過引導，稍後手動配置）
& ([scriptblock]::Create((Get-Content -Raw "$env:TEMP\oc-audit\install.ps1"))) -NoOnboard
```

> 實測結論：官方安裝器只在**用戶空間**操作——更新用戶 PATH、在自己的安裝目錄刪除舊文件，**不**寫 `HKLM/HKCU` 註冊表、不碰 `system32`、不建服務、不加開機自啟（gateway 是以「登入項/Scheduled Task」形式，且由後續 `daemon` 命令顯式安裝）。完整審計結論已沉澱在本流程的排障與安全章節。

## 安裝後做什麼

- **不要**急著跑 `openclaw onboard`（引導）——本流程採「手動最小配置」，每一步都看得見、可控，更符合安全取向。
- 進入 [`03-configure-deepseek.md`](03-configure-deepseek.md) 配置模型。

## 卸載（若需要）

```powershell
openclaw daemon stop        # 先停 gateway
openclaw daemon uninstall   # 移除登入項/計劃任務
# 再刪除配置目錄（可選，會清掉密鑰與會話）：
#   C:\Users\<你>\.openclaw
```
