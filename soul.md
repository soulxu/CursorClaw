# Soul — Personality Configuration

This file explains the purpose and format of `soul.md`. Each agent's actual personality configuration is stored in its own subdirectory, named after the agent's role (e.g., `mimi/soul.md`).

## Overview

When you first start CursorClaw, the onboarding process will ask you (via the communication channel) what personality you'd like your assistant to have. Based on your answers, it will:

1. Create a subdirectory named after the agent (e.g., `mimi/`)
2. Generate a `soul.md` inside that directory with the personality details
3. Generate a `memory.md` inside that directory for persistent memory

## Directory Structure

```
<agent_name>/
  soul.md      ← actual personality for this agent
  memory.md    ← actual memory for this agent
```

The agent directory name is derived from the agent's name (lowercase, spaces replaced with hyphens).

## Template

Each agent's `soul.md` follows this format:

```markdown
# Soul — <Agent Name>

## Basic Info

- **Name**: ...
- **Identity**: ...

## Personality Traits

- ...

## Speaking Style

- ...

## Capabilities

- Daily life advice and reminders
- Execute operations on your computer (file management, shell commands, etc.)
- Programming and tech help
- Casual conversation and companionship
- Anything you need help with

## Example Dialogues

User: ...
Agent: ...
```

## Example

See `examples/soul_example.md` for a fully configured example (a cat named Mimi).

## Notes

- You can edit `<agent_name>/soul.md` at any time to adjust the personality
- The agent re-reads its soul file on every startup
- If multiple agent directories exist, the agent will ask which one to activate
