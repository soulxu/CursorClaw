# 通讯渠道：iMessage

本文件描述 CursorClaw 的 iMessage 渠道实现，使用 [`imsg`](https://github.com/steipete/imsg) CLI 工具。

## 前置要求

- macOS 14.0 (Sonoma) 或更高版本
- Mac 上已登录并正常使用 iMessage
- 在系统设置 → 隐私与安全 → 完全磁盘访问权限中授予 Cursor
- 安装 `imsg` CLI：

```bash
brew install steipete/tap/imsg
```

验证安装：

```bash
imsg chats
```

## 所需 Allowlist 命令

添加到 `~/.cursor/cli-config.json` → `permissions.allow`：

- `Shell(imsg)` — iMessage 操作
- `Shell(sips)` — 图片格式转换（用于 iMessage 附件）

## 初始化参数

以下参数需要在初始化渠道前从用户处获取（在 Cursor 界面中）：

| 参数 | 说明 | 示例 |
|---|---|---|
| **通讯对象** | 要沟通的用户的手机号或 Apple ID | `+8613800138000` 或 `user@icloud.com` |

## 渠道初始化

用户提供所需参数后，按以下步骤初始化渠道：

1. 向通讯对象发送问候消息：`imsg send --phone <通讯对象> --text "<问候语>"`
2. 运行 `imsg chats` 查找新创建的对话及其 `chat_id`
3. 保存 `chat_id` — 后续所有通讯都使用此 ID
4. 在此 `chat_id` 上启动后台监听

## 命令参考

### 发送消息（已有对话）

```bash
imsg send --chat-id <CHAT_ID> --text "<消息内容>"
```

向已有对话发送文本消息。

### 发送消息（新联系人）

```bash
imsg send --phone <手机号> --text "<消息内容>"
```

向指定手机号发送消息。如果对话不存在，iMessage 会自动创建。渠道初始化时使用。

### 列出对话

```bash
imsg chats
```

列出所有 iMessage 对话。在发送首条问候消息后使用此命令查找新创建对话的 `chat_id`。

### 获取历史消息

```bash
imsg history --chat-id <CHAT_ID> --limit <N> --json
```

获取对话中最近的 N 条消息。正常启动时用来获取最新消息的 `rowid`。

### 监听（后台监控）

```bash
imsg watch --chat-id <CHAT_ID> --json --since-rowid <LAST_ID> --debounce 250ms
```

实时监听对话中的新消息。必须以 `is_background: true` 启动。输出流写入终端文件，由轮询循环读取。

如果 watch 进程意外退出，需自动重启。

## 轮询回复

引导设置的提问过程中，每个问题：

1. 通过 `imsg send --chat-id <CHAT_ID> --text "..."` 发出问题
2. 进入小型轮询循环（读取 watch 终端输出，等待用户新消息）
3. 获取回复后再问下一个问题

## 消息识别

这是一个 **自聊天（self-chat）** 场景（用户给自己发消息），所有消息都会出现两次：

- `is_from_me: true` — 发送方副本
- `is_from_me: false` — 接收方副本

**区分用户消息和机器人消息的方法：**

- 通过 `imsg send` 发出的消息，其 `text` 字段在 `is_from_me: true` 时会带有特殊字节前缀（`\ufffd` 字符）
- 用户手动输入的消息没有这个前缀
- **主要规则：只处理 `is_from_me: true` 且没有 `\ufffd` 前缀的消息**

**会话内追踪（补充机制）：**

仅靠 `\ufffd` 前缀在跨会话场景下可能不够可靠。Agent 应在内存中维护当前会话发送过的消息文本集合。处理 `is_from_me: true` 消息时：

1. 消息文本（去掉 `\ufffd` 及控制字符后）匹配发送集合 → 机器人消息，跳过
2. 有 `\ufffd` 前缀且不在发送集合中 → 可能是上一个会话的机器人消息，跳过
3. 无 `\ufffd` 前缀且不在发送集合中 → 用户消息，处理

**启动时检查未回复消息：**

正常启动时（开始 watch 之前），获取最近的消息历史后，向前扫描最近的消息，找出最后一条机器人回复之后是否还有未回复的用户消息（即 `is_from_me: true` 且无 `\ufffd` 前缀的消息）。如果有，先处理这些消息再启动 watch 循环。这样可以防止 Agent 会话重启时丢失用户消息。

## 渠道专属配置

以下值在渠道初始化后保存到 `<AGENT_DIR>/memory.md` 的 **「Channel: imessage」** 章节：

- **Chat ID** — iMessage 对话标识符，在渠道初始化时自动创建
