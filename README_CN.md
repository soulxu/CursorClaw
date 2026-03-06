# CursorClaw

**把 Cursor CLI 变成 [OpenClaw](https://github.com/anthropics/openclaw) (ClawBot) 式的 AI 个人助理。**

CursorClaw 将 [Cursor](https://cursor.sh) 的 CLI Agent 变成一个类似 OpenClaw (ClawBot) 的持续运行的 AI 个人助理。它通过特定的 **通讯工具** 与你沟通——**iMessage** 就是其中之一（通过 `imsg`）。你通过这些通讯工具发出指令，Agent 接收后在 **你的电脑上执行命令、完成任务**。未来会支持更多渠道。目前支持在 Mac 上运行。

## 功能特性

- **通讯方式** — 当前支持 iMessage（通过 `imsg`）：实时监听对话并自动回复
- **自定义人格** — 你来定义助理的性格：可爱的猫咪、正式的管家、幽默的海盗……随你喜欢
- **持久记忆** — 跨会话记住你告诉它的一切
- **定时任务** — 每日提醒、天气播报、股价查询、早安晚安问候
- **自动引导** — 无需编辑配置文件，助理通过 iMessage 聊天完成初始设置

## 环境要求

| 要求 | 详情 |
|---|---|
| **macOS** | 14.0 (Sonoma) 或更高版本 |
| **Cursor** | 最新版本，支持 Agent 模式 |
| **imsg CLI** | `brew install steipete/tap/imsg` |
| **完全磁盘访问权限** | 在系统设置 → 隐私与安全 → 完全磁盘访问权限中授予 Cursor |
| **iMessage** | 在 Mac 上已登录并正常使用 |

## 快速开始

### 1. 克隆 CursorClaw

```bash
git clone https://github.com/anthropics/CursorClaw.git
cd CursorClaw
```

### 2. 启动 Agent

在 `CursorClaw` 目录下运行：

```bash
agent "读取 instruction_cn.md 并启动"
```

或英文版：

```bash
agent "Read instruction.md and start"
```

Agent 会自动：
- 读取指令文件
- 检测到这是首次运行
- 列出你的 iMessage 聊天列表，请你选择要监听的对话
- 通过 iMessage 向你发送引导设置问题
- 根据你的回答构建人格和记忆
- 开始监听和自动回复

### 3. 搞定！

你的 AI 助理已经在 iMessage 上线了。像跟朋友聊天一样跟它说话吧。

## 工作原理

当前（iMessage 渠道）：

```
你 (iMessage) <──> imsg CLI <──> Cursor Agent (读取 instruction.md)
                                      │
                                      ├── soul.md    (人格设定)
                                      └── memory.md  (长期记忆)
```

Agent 使用 [`imsg`](https://github.com/steipete/imsg) CLI 工具监听 iMessage 新消息（当前渠道），在 Cursor Agent 模式中运行轮询循环，读取新消息、根据人格设定处理并回复——完全自主运行。

## 自定义

### 人格设定 (`soul.md`)

引导设置完成后，助理的人格保存在 `soul.md` 中。你可以随时编辑：
- 名字和角色
- 说话风格和语气
- 使用的语言
- 表情符号的使用
- 口头禅或特殊习惯

### 记忆 (`memory.md`)

Agent 会把你要求它记住的一切写入 `memory.md`，按分类整理：
- 你的个人信息和偏好
- 定时任务和提醒
- 重要日期
- 其他你告诉它记住的事情

### 定时任务

告诉你的助理：
- *"每天早上8点提醒我吃药"*
- *"每天早上7点给我报天气"*
- *"每天晚上10点跟我说晚安"*

任务存储在 `memory.md` 中，在 ±5 分钟的时间窗口内自动执行。

## 架构

Agent 运行一个简洁有效的循环：

```
┌─────────────────────────────────┐
│           初始化                │
│  读取 instruction/soul/memory   │
│  启动后台 `imsg watch`          │
└──────────────┬──────────────────┘
               │
               ▼
┌─────────────────────────────────┐
│          轮询循环               │◄──┐
│  1. Sleep (白天5s / 夜间120s)   │   │
│  2. 检查时间 → 执行定时任务     │   │
│  3. 读取 watch 输出             │   │
│  4. 识别新消息                  │   │
│  5. 处理并回复                  │   │
│  6. 按需更新记忆                │   │
└──────────────┬──────────────────┘   │
               │                      │
               └──────────────────────┘
```

## 消息识别

CursorClaw 使用自聊天模式（给自己发消息）。`imsg` 工具发送的消息带有特殊字节前缀（`\ufffd`），用于区分：

- **你的消息**：`is_from_me: true`，没有前缀 → 处理并回复
- **机器人消息**：`is_from_me: true`，有 `\ufffd` 前缀 → 忽略

## 常见问题

**Q: 支持群聊吗？**
A: 目前仅支持一对一自聊天。群聊支持可能在未来加入。

**Q: Cursor 在后台运行时也能工作吗？**
A: 可以，只要 Cursor 在运行且 Agent 会话处于活跃状态。

**Q: 怎么停止 Agent？**
A: 在 Cursor Agent 聊天中按停止按钮，或关闭聊天。

**Q: 会消耗我的 API 额度吗？**
A: 会的。Agent 使用你在 Cursor 中配置的 LLM（Claude、GPT 等）。

## 贡献

欢迎贡献！请随时提 Issue 或提交 Pull Request。

## 许可证

[MIT](LICENSE)

## 致谢

- [imsg](https://github.com/steipete/imsg) by [@steipete](https://github.com/steipete) — 让 iMessage 自动化成为可能的 CLI 工具
- [Cursor](https://cursor.sh) — 驱动 Agent 的 AI 代码编辑器
