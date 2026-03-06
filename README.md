# CursorClaw

**Turn Cursor CLI into an [OpenClaw](https://github.com/anthropics/openclaw) (ClawBot)-style AI personal assistant.**

CursorClaw transforms [Cursor](https://cursor.sh)'s CLI agent into a persistent, always-on AI personal assistant — similar to OpenClaw (ClawBot). It talks to you through specific **communication tools** — **iMessage** is one of them (via `imsg`). You send instructions through these tools; the agent receives them and **runs commands and completes tasks on your computer**. More channels may be added in the future. Currently supports running on Mac.

## What It Does

- **Communication** — Currently supports iMessage (via `imsg`): monitors your chat and responds in real-time
- **Custom Personality** — You define who your assistant is: a friendly cat, a formal butler, a witty pirate — anything
- **Persistent Memory** — Remembers what you tell it across sessions
- **Scheduled Tasks** — Daily reminders, weather reports, stock prices, good-morning messages
- **Self-Onboarding** — No config files to edit; the agent sets itself up by chatting with you

## Prerequisites

| Requirement | Details |
|---|---|
| **macOS** | 14.0 (Sonoma) or later |
| **Cursor** | Latest version with Agent mode |
| **imsg CLI** | `brew install steipete/tap/imsg` |
| **Full Disk Access** | Grant to Cursor (System Settings → Privacy & Security → Full Disk Access) |
| **iMessage** | Signed in and working on your Mac |

## Quick Start

### 1. Clone CursorClaw

```bash
git clone https://github.com/anthropics/CursorClaw.git
cd CursorClaw
```

### 2. Start the Agent

Run the following command in the `CursorClaw` directory:

```bash
agent "Read instruction.md and start"
```

The agent will:
- Read its instructions
- Detect this is a first run
- List your iMessage chats and ask you to pick one
- Send you onboarding questions via iMessage
- Build its personality and memory based on your answers
- Start monitoring and responding

### 3. That's It

Your AI assistant is now live on iMessage. Talk to it like you would a friend.

## How It Works

Currently (iMessage channel):

```
You (iMessage) <──> imsg CLI <──> Cursor Agent (reads instruction.md)
                                      │
                                      ├── soul.md    (personality)
                                      └── memory.md  (long-term memory)
```

The agent uses the [`imsg`](https://github.com/steipete/imsg) CLI tool to watch for incoming iMessages (current channel). It runs a polling loop inside Cursor's agent mode, reading new messages, processing them according to its personality, and replying — all autonomously.

## Customization

### Personality (`soul.md`)

After onboarding, your assistant's personality is saved in `soul.md`. You can edit it anytime to change:
- Name and character
- Speaking style and tone
- Language
- Emoji usage
- Any quirks or catchphrases

### Memory (`memory.md`)

The agent stores everything you ask it to remember in `memory.md`, organized by category:
- Your personal info and preferences
- Scheduled tasks and reminders
- Important dates
- Anything else you tell it to remember

### Scheduled Tasks

Tell your assistant things like:
- *"Remind me to take medicine every day at 8am"*
- *"Send me a weather report every morning at 7am"*
- *"Wish me good night at 10pm"*

Tasks are stored in `memory.md` and automatically executed within a ±5 minute window.

## Architecture

The agent runs a simple but effective loop:

```
┌─────────────────────────────────┐
│         Initialization          │
│  Read instruction/soul/memory   │
│  Start background `imsg watch`  │
└──────────────┬──────────────────┘
               │
               ▼
┌─────────────────────────────────┐
│         Polling Loop            │◄──┐
│  1. Sleep (5s day / 120s night) │   │
│  2. Check time → run tasks     │   │
│  3. Read watch output           │   │
│  4. Identify new messages       │   │
│  5. Process & reply             │   │
│  6. Update memory if needed     │   │
└──────────────┬──────────────────┘   │
               │                      │
               └──────────────────────┘
```

## Message Identification

CursorClaw uses a self-chat pattern (you message yourself). The `imsg` tool adds a special byte prefix (`\ufffd`) to messages it sends, so the agent can distinguish:

- **Your messages**: `is_from_me: true`, no prefix → process and respond
- **Bot messages**: `is_from_me: true`, has `\ufffd` prefix → ignore

## FAQ

**Q: Does this work with group chats?**
A: Currently designed for 1-on-1 self-chat. Group chat support may come in the future.

**Q: Does it work when Cursor is in the background?**
A: Yes, as long as Cursor is running and the agent session is active.

**Q: How do I stop the agent?**
A: Press the Stop button in the Cursor agent chat, or close the chat.

**Q: Can I run multiple assistants?**
A: You could run multiple instances with different chat-ids, but this is not officially supported yet.

**Q: Does it use my API credits?**
A: Yes — the agent runs on whatever LLM you have configured in Cursor (Claude, GPT, etc.).

## Contributing

Contributions are welcome! Feel free to open issues or submit pull requests.

## License

[MIT](LICENSE)

## Acknowledgments

- [imsg](https://github.com/steipete/imsg) by [@steipete](https://github.com/steipete) — the CLI tool that makes iMessage automation possible
- [Cursor](https://cursor.sh) — the AI-powered code editor that powers the agent
