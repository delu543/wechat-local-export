# 微信本地数据提取与每日日报助手（WeChat Local Export）

> **当前版本：`0.3.0-dev.1`** ｜ 适配 **GrokBot / Codex / Claude Code** 等本地 AI 代理  
> 一句话说明：**纯本地只读读取你本机微信 4.x 的聊天数据**——支持一键生成**每日微信情报日报（`daily-digest`）**、按“聊天名 + 时间段”扫描归档文字/图片/语音/文件，以及**语音按顺序直接合成 MP4**。  
> **绝不操控微信界面 · 不改原版微信 · 不关闭系统 SIP · 数据不出本机。**

---

## 一眼看懂：这个项目能帮你做什么？

| 核心能力 | 一句话效果 | 典型用法 |
| --- | --- | --- |
| **每日微信日报（`0.3.0` 新增）** | 一次在线刷新，自动汇总你所有**置顶聊天**（或自定义清单）的新消息，带增量游标，不重复读旧消息 | 搭配 GrokBot 定时任务，每晚自动总结全天 `@我`、重点群聊精华和待办 |
| **语音一键合成 MP4** | 把某个群/某段时间的多条语音按说话顺序直接流式合并成一个 MP4 | 老板/老师连发 20 条长语音，合成一条倍速听 |
| **精准聊天扫描与归档** | 按“聊天名称 + 绝对时间范围 + 内容类型”先只读扫描报条数，你确认后再导出 | 归档项目群资料、导出文字/图片/语音/普通文件 |
| **不关微信、在线快照** | 利用 macOS APFS 写时复制秒级拍只读快照，平时微信照常开着，自动清理旧快照（默认留 1 份） | 每天定时跑完全不打扰日常聊天，也不爆硬盘 |

---

## 最简单的上手方式（推荐：直接把提示词发给 AI 代理）

你**不需要自己敲一堆复杂命令**。本仓库自带写给 AI 代理看的完整搭建指南（[GUIDE.md](GUIDE.md) 与 [AGENTS.md](AGENTS.md)），把下面对应系统的提示词直接复制发给 **GrokBot / Codex / Claude Code**，需要你点头授权或登录微信时，它会一步步提示你。

### 方式 A：macOS 用户（Apple Silicon M 芯片 + 官方微信 4.1.x，实机已完整验证）

#### 第 0 步：先做 2 个 1 分钟准备
1. 打开官方 Mac 微信并登录当前账号（点开任意聊天即可，保持微信打开）；把想每天分析的重点群/联系人顺手在微信里**置顶**。
2. 打开 **系统设置 → 隐私与安全性 → 完全磁盘访问权限**，勾选你使用的 **Grok Bot**（或 Codex / 终端），如有提示则重启对应应用。

#### 第 1 步：复制下面提示词发给 GrokBot / Codex，自动完成安装

```text
请帮我在本机安装并检查微信本地数据提取项目，仓库地址：
https://github.com/delu543/wechat-local-export

请你按以下顺序执行：
1. 把仓库克隆到本机干净目录（若已有本地目录先检查，不要覆盖已有改动）。
2. 完整阅读根目录的 GUIDE.md 和 AGENTS.md，严格遵守安全边界（只读、不操控微信界面、不改原版微信、不上传聊天内容）。
3. 先运行 ./scripts/codex_bootstrap.sh doctor 检查环境；环境满足后运行 ./scripts/codex_bootstrap.sh install 完成私有运行环境、Swift 工具和 Skill 安装。
4. 安装完成后停下来向我汇报结果，不要自动进入微信密钥初始化阶段，等我下一步指令。
```

> 安装器会把所有依赖装到私有目录 `~/Library/Application Support/WeChatLocalExport/`，不污染系统 Python，并安装 `wechat-local-export` 和 `wechat-local-export-setup` 两个 Skill。

#### 第 2 步：每个微信账号做一次“密钥初始化”（仅需一次）

确认官方微信已登录并打开任意聊天后，发给 AI：

```text
我已登录官方 Mac 微信并打开了任意聊天窗口，请显式执行 $wechat-local-export-setup 完成一次性本机初始化。
每一步涉及安装初始化依赖、复制微信副本取密钥、以及保留本机私有解密快照时，都单独停下来告诉我并等我回复“同意”再继续；完成后清理副本，我重新打开原版微信后再跑一次只读 doctor 确认就绪。
```

> **为什么安全？** 初始化只对**临时副本**操作，原版微信纹丝不动，无需关闭 SIP；拿到本地数据库密钥后存入仅本人可读的 `0600` 私有文件，绝不进入日志、对话或网络。每个新微信账号都需要单独初始化一次，历史账号目录不会显示为可选项；程序只接受当前官方微信进程能够唯一证明的账号绑定，不会按文件时间、目录顺序或大小猜测账号。之后日常使用彻底告别副本，只要开着原版微信即可。

#### 第 3 步：试跑每日微信日报（`daily-digest`）

```text
请用 wechat-local-export 客户端的 daily-digest 功能先试跑一次（加上 --dry-run，不推进游标），读取我所有置顶聊天从今天 08:00 到现在的新消息，并按以下 5 个板块生成《今日微信情报日报》：
1. 🔴 紧急 / @我 & 待我回复
2. 📌 重点群聊精华（过滤水群闲聊，只提炼结论、通知和重要资源）
3. 💬 重要私聊与我今天的承诺
4. 📊 今日最活跃群聊与高频话题
5. ✅ 优先行动清单（最多 3～5 条）
```

试跑满意后，对 GrokBot 说一句：**“把这套流程存成 Skill，把游标初始化到现在，再建一个每天 21:30 自动跑正式版 `daily-digest`（不加 `--dry-run`）并汇报日报的定时任务。”** 即可每天到点自动收日报。

---

### 方式 B：Windows 10/11 x64 用户（源码预览）

> Windows 目前为原生源码预览版，自动化测试已通过，但尚未完成官方 Windows 微信全版本实机验收；详见 [Windows 说明与验证边界](docs/WINDOWS_SUPPORT.md) 与 [Windows 新手指南](docs/WINDOWS_FIRST_RUN.md)。

在 Windows 电脑的 **Codex / AI 本地任务**中发送（不要在微信里发送）：

```text
我用 Windows 10/11，请帮我在本机安装并使用这个项目：
https://github.com/delu543/wechat-local-export
请取得完整源码，读取 AGENTS.md 和 docs/WINDOWS_FIRST_RUN.md，按 Windows 新手流程检查和安装。
需要我本人登录微信或确认隐私操作时，一次明确告诉我下一步；其余能安全代办的步骤由你完成。
如果本地已有这个项目，先检查，不要覆盖已有文件或改动其他项目。
```

---

## 日常直接对话怎么用？

初始化完成后，保持官方微信打开，平时用自然语言就能指挥 AI：

```text
跑一次今天的 daily-digest，把所有置顶群的新消息整理成一页日报。
```

```text
把“示例讨论群”2026 年 7 月 24 日 14:00 到 15:30 的所有语音直接合成 MP4。
```

```text
先扫描“项目讨论群”昨天 9:00 到 18:00 的文字、图片和文件，告诉我各有多少条。
```

```text
归档“家人群”本周一到周三的文字、图片、语音和文件。
```

> **提示**：不需要截图、不需要自己找数据库文件、也不需要手动翻历史记录或逐条点语音。群名重名或有歧义时，工具会停止并列出候选让你选择，绝不瞎猜。

---

## 支持导出与分析哪些内容？

| 类型 | 当前能力 |
| --- | --- |
| **每日多聊天日报（`daily-digest`）** | 一次刷新读取所有置顶聊天（或 `--chats-file` 自定义清单），含发言数、Top 5 发言人、`@我`、本人发言与核心文字；增量游标防重复，微信关闭时安全回退上次快照并标记 `stale` |
| **文字 / 系统消息** | 导出结构化 JSON、JSONL、Markdown 和 HTML（`0.3.0` 支持通过 `Name2Id` 解析 1:1 聊天与本人群发言的真实显示名） |
| **图片** | 支持 V1 / V2 / 旧 XOR 解密，WXGF 转 JPEG，优先选择最高可验证质量 |
| **语音** | 导出 SILK、逐条 M4A，或一次流式编码直接合并为 H.264 + AAC MP4 |
| **普通文件** | 仅导出消息元数据能够唯一验证的本机已下载文件 |
| **表情** | 导出可唯一验证的本机明文资源；私有加密格式保留元数据 |
| **视频 / 公众号** | 视频当前仅导出元数据（不含视频本体）；公众号消息不在 `message_N` 表中，日报会以单聊天提示列出 |

*注：仅限读取并导出本机实际已保存的数据；新电脑尚未同步或已清理的云端附件无法凭空恢复。*

---

## 手动命令行安装与卸载（给开发者 / 进阶用户）

### macOS

需要 macOS 15+、官方 Mac 微信 4.x、系统 `/usr/bin/python3`（3.9–3.13）及 Xcode Command Line Tools（`xcode-select --install`）：

```bash
git clone https://github.com/delu543/wechat-local-export.git
cd wechat-local-export
./scripts/codex_bootstrap.sh doctor
./scripts/codex_bootstrap.sh install
```

卸载对话层（仅移除本项目管理的 Skill 与 `current` 软链，不删你的私有快照、密钥和导出文件）：

```bash
./scripts/codex_bootstrap.sh uninstall
```

### Windows 10/11 x64（源码预览）

需要官方微信 4.x、CPython 3.12 x64 和 VC++ 运行库：

```powershell
.\scripts\codex_bootstrap.ps1 doctor
.\scripts\codex_bootstrap.ps1 install
```

### `daily-digest` 常用命令速查

```bash
# 从各聊天游标（首次默认近 24 小时）读到现在并推进游标
python3 <client> daily-digest

# 试跑一次，不推进游标
python3 <client> daily-digest --dry-run --since "2026-11-22 08:00"

# 只把所有聊天游标初始化到指定时间点
python3 <client> daily-digest --init-cursors-at "2026-11-22 21:00"

# 使用私有清单文件指定监控的群聊/联系人（不依赖微信置顶）
python3 <client> daily-digest --chats-file /path/to/focus-chats.txt
```

---

## 隐私、安全与验证边界

- **只读、不碰界面**：无键盘输入、无鼠标点击、无发消息接口，不需要屏幕录制、辅助功能或麦克风权限。
- **零上传设计**：本地 helper 无遥测、无远程 MCP、无上传接口；普通导出不把正文写入日志或命令行。*（注：当你主动要求 AI 模型阅读本地日报 JSON 做总结时，选中的文本会按你所用 AI 账号的隐私设置进入模型推理链路，请知悉。）*
- **最小权限存储**：macOS 私有目录 `0700`、密钥与日报文件 `0600`（日报默认 **14 天自动清理**，在线快照默认只保留最新 **1 份**）；Windows 采用当前用户与 SYSTEM 专属 ACL + DPAPI 加密。
- **验证边界**：
  - ✅ 已实机验证：Apple Silicon（arm64）Mac、SIP 保持开启、官方 Mac 微信 4.1.x；文字扫描、语音 MP4、每日日报 `daily-digest`。
  - ⚠️ 源码预览 / 未完整实机验收：Windows 10/11 x64、Intel Mac、微信 3.x 或其他未验证的 4.x 小版本。
- **声明**：本项目为开发预览源码与 Skill（当前许可证状态 `UNLICENSED`，仅限学习与处理本人有权访问的本机数据，不代表商用或二次分发授权），与腾讯、微信、OpenAI、xAI 均无官方隶属关系。

---

## 回归测试与完整文档导航

安装依赖后可运行全套本地校验（隐私扫描 + Skill 校验 + 全量单测 + Swift 构建与自检）：

```bash
./scripts/release_check.sh
```

更多技术细节请参阅：

- [AI 代理完整搭建指南（GUIDE.md）](GUIDE.md) — 面向 GrokBot / Codex / Claude Code 的完整技术手册
- [版本更新日志（CHANGELOG.md）](CHANGELOG.md) — `0.3.0-dev.1` 更新明细
- [AI 代理操作约束（AGENTS.md）](AGENTS.md) — 安装与交互安全边界
- [产品架构（docs/PRODUCT_ARCHITECTURE.md）](docs/PRODUCT_ARCHITECTURE.md) ｜ [能力矩阵（docs/CAPABILITY_MAP.md）](docs/CAPABILITY_MAP.md)
- [Windows 新手指南](docs/WINDOWS_FIRST_RUN.md) ｜ [Windows 支持边界](docs/WINDOWS_SUPPORT.md)
- [隐私边界（PRIVACY.md）](PRIVACY.md) ｜ [安全模型（SECURITY.md）](SECURITY.md) ｜ [第三方组件声明（THIRD_PARTY_NOTICES.md）](THIRD_PARTY_NOTICES.md)
