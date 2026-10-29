# OpenClaw + Telegram 部署后参考手册（给人看的）

> 版本 v1.1.0 · 适用于 OpenClaw 2026.10.8 · Windows 10/11 · DeepSeek
> 这本手册是「装好之后」给**人类**日常翻阅用的。所有命令都能在任意 PowerShell 窗口直接执行，**不需要先 `cd` 到哪个文件夹**。

---

## 0. 先记住三句话

1. **命令不用 cd**：`openclaw` 是全局命令，任何位置都能跑（跟 Claude Code 要先进入项目文件夹不同）。
2. **东西都在 `C:\Users\<你>\.openclaw\`**：配置、密钥、agent 工作区、记忆全在这个文件夹。
3. **关控制台 ≠ 关 bot**：关掉浏览器/终端，bot 照样在后台回复；真要停，用 `openclaw daemon stop`。

---

## 1. 位置速查表

| 要找什么 | 位置 |
|---|---|
| 配置文件 | `C:\Users\<你>\.openclaw\openclaw.json` |
| 密钥 / 凭据 / 状态 | `C:\Users\<你>\.openclaw\state\openclaw.sqlite` |
| agent 工作区（它读写文件的地方） | `C:\Users\<你>\.openclaw\workspace\` |
| agent 记忆 / 会话 | `C:\Users\<你>\.openclaw\agents\main\agent\openclaw-agent.sqlite` |
| 日志 | `C:\Users\<你>\.openclaw\logs\` |

> 把 `<你>` 换成你的 Windows 用户名。最快的办法：在 PowerShell 输入 `echo $env:USERPROFILE`。

### 确定 / 限制 agent 工作区

- **确定**：`openclaw config get agents.defaults.workspace`——若返回「unset，runtime default applies」，默认就是 `~\.openclaw\workspace\`。
- **限制**：默认 agent 只在 `~\.openclaw\workspace\` 内活动、不碰你其他磁盘的文件。要换位置：`openclaw config set agents.defaults.workspace "D:\某个文件夹"` + `openclaw gateway restart`。**别把敏感文件放进 `workspace\`**，agent 读得到。

---

## 2. 日常操作速查

| 你想做的事 | 命令 / 做法 |
|---|---|
| 查状态（最重要，先跑这个） | `openclaw status` |
| **开** bot | `openclaw daemon start` |
| **关** bot | `openclaw daemon stop` |
| 改了配置后重启 | `openclaw gateway restart` |
| 看实时日志（排障） | `openclaw logs --follow`（Ctrl+C 退出） |
| 测模型通不通 | `openclaw agent --message "hi"` |
| 打开控制台（网页） | `openclaw dashboard` |
| 打开控制台（终端） | `openclaw tui` |
| 关闭控制台 | 关浏览器标签页 / TUI 内 Ctrl+C |

> **重启电脑后**：登录 Windows，等 1–2 分钟，`openclaw status` 看到 Gateway `running`、Telegram `ON / OK` 即正常；没起来就 `openclaw daemon start`。

---

## 3. 验收清单（怎么算「装好了」）

在 Telegram 私信 bot 一个需要思考的问题（例：`用中文解释为什么天空是蓝色的，并给三个关键词`），确认：

1. ✅ 能**正常回复**（不是「Couldn't sign in…」）。
2. ✅ **只看得到最终答案**，看不到思考/推理/工具调用。
3. ✅ 能**快速开/关控制台**（`openclaw dashboard` → 关标签页）。

---

## 4. 新手注意事项

- 🔑 **密钥/token 绝不外泄**：DeepSeek key、bot token 等同密码，勿贴公开仓库/群聊/截图。
- 👥 **白名单只加自己**：勿把 `dmPolicy` 改成 `open`，否则知道 bot 用户名的人都能用你的余额。
- 🏠 **别动 `gateway.bind`**：保持 `loopback`，勿改 lan/公网。
- 💰 **DeepSeek 按量付费**：留意余额。
- 🔄 **改配置后要重启**：`openclaw gateway restart` 才生效。
- 🖥️ **关控制台 ≠ 停 bot**（见验收清单第 3 条）。
- 😴 **睡眠/注销会停 bot**：想 24×7 常驻，关闭系统睡眠。
- 📁 **别把敏感文件放进 `~\.openclaw\workspace\`**：agent 读得到那里面。
- ✅ **首次 UAC 弹窗是 Node.js 升级**：正常，点「是」。

---

## 5. 常见问题速查

### 「⚠️ Couldn't sign in to the AI service」
密钥没进后台进程（通常是只设了环境变量）。解决办法：

```powershell
openclaw models auth paste-api-key --provider deepseek
openclaw gateway restart
```

### 「Config invalid: Unrecognized key: suppressNarrationOnMessaging」
这个键不存在，删掉即可。隐藏思考用 `reasoningDefault off`。

### bot 收到消息但不回复
- 先 `openclaw pairing list` 看有没有待批准的配对码。
- 再 `openclaw config get channels.telegram` 确认 `allowlist` + 你的 ID 在 `allowFrom` 里。

### status 有 Telegram 但收发不了（中国大陆）
```powershell
curl.exe -s -o NUL -w "%{http_code}" https://api.telegram.org
```
不通就是代理没生效，让整台电脑走代理后 `openclaw gateway restart`。

---

## 6. 安全要点

- 密钥用凭据库（`paste-api-key`），不落 `openclaw.json` 明文。
- gateway 只绑 `loopback`（127.0.0.1），不暴露。
- Telegram 白名单：只有你的数字 ID 能指挥 bot。
- 备份 = 复制整个 `~\.openclaw\`（备份含密钥，当机密对待）。

---

> 完整部署流程与排障详解，见同仓库的 `docs/` 系列文档。
