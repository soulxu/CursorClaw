# Communication Channel: iMessage

This file describes the iMessage channel implementation for CursorClaw, using the [`imsg`](https://github.com/steipete/imsg) CLI tool.

## Prerequisites

- macOS 14.0 (Sonoma) or later
- iMessage signed in and working on your Mac
- Full Disk Access granted to Cursor (System Settings → Privacy & Security → Full Disk Access)
- Install the `imsg` CLI:

```bash
brew install steipete/tap/imsg
```

Verify it works:

```bash
imsg chats
```

## Required Allowlist Commands

Add these to `~/.cursor/cli-config.json` → `permissions.allow`:

- `Shell(imsg)` — iMessage operations
- `Shell(sips)` — image format conversion (for iMessage attachments)

## Setup Parameters

The following parameters must be collected from the user (in Cursor) before the channel can be initialized:

| Parameter | Description | Example |
|---|---|---|
| **Recipient** | The phone number or Apple ID of the user to communicate with | `+1234567890` or `user@icloud.com` |

## Channel Initialization

After the user provides the required parameters, initialize the channel by following these steps:

1. Send a greeting to the recipient: `imsg send --phone <RECIPIENT> --text "<GREETING>"`
2. Run `imsg chats` to find the newly created conversation and its `chat_id`
3. Save the `chat_id` — all future communication uses this ID
4. Start the background watch on this `chat_id`

## Command Reference

### Send Message (to existing chat)

```bash
imsg send --chat-id <CHAT_ID> --text "<MESSAGE>"
```

Sends a text message to an existing conversation.

### Send Message (to new recipient)

```bash
imsg send --phone <PHONE_NUMBER> --text "<MESSAGE>"
```

Sends a text message to a phone number. If no conversation exists yet, iMessage creates one automatically. Used during channel initialization.

### List Chats

```bash
imsg chats
```

Lists all iMessage conversations. Used after sending the first greeting to find the newly created chat's `chat_id`.

### Get History

```bash
imsg history --chat-id <CHAT_ID> --limit <N> --json
```

Retrieves the most recent N messages from a conversation. Use on normal startup to get the latest message `rowid`.

### Watch (Background Monitoring)

```bash
imsg watch --chat-id <CHAT_ID> --json --since-rowid <LAST_ID> --debounce 250ms
```

Monitors a conversation for new messages in real-time. Must be launched with `is_background: true`. Output streams to a terminal file that the polling loop reads.

If the watch process dies, restart it automatically.

## Polling for Replies

During onboarding questions, for each question:

1. Send the question via `imsg send --chat-id <CHAT_ID> --text "..."`
2. Enter a mini polling loop (read the watch terminal output, wait for a new user message)
3. Get the reply before asking the next question

## Message Identification

This is a **self-chat** scenario (the user messages themselves). All messages appear twice:

- `is_from_me: true` — sender copy
- `is_from_me: false` — receiver copy

**Distinguishing user messages from bot messages:**

- Messages sent via `imsg send` have a special byte prefix (`\ufffd` character) in the `text` field when `is_from_me: true`
- Messages typed manually by the user do NOT have this prefix
- **Primary rule: only process messages where `is_from_me: true` AND there is NO `\ufffd` prefix**

**In-session tracking (supplementary mechanism):**

The `\ufffd` prefix alone may not be fully reliable across agent sessions. The agent should maintain an in-memory set of message texts sent via `imsg send` during the current session. When evaluating `is_from_me: true` messages:

1. Message text (after stripping `\ufffd` and control characters) matches the sent set → bot message, skip
2. Has `\ufffd` prefix and NOT in the sent set → likely a bot message from a previous session, skip
3. No `\ufffd` prefix and NOT in the sent set → user message, process

**Startup check for unresponded messages:**

On normal startup (before starting the watch), after retrieving recent message history, scan backwards through the messages to find any user messages (`is_from_me: true` without `\ufffd` prefix) that appear after the last bot reply. If found, process and respond to these messages before starting the watch loop. This prevents user messages from being lost across agent session restarts.

## Channel-Specific Configuration

These values are stored in `<AGENT_DIR>/memory.md` under the **"Channel: imessage"** section after setup:

- **Chat ID** — the iMessage conversation identifier, created during channel initialization
