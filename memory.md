# Long-Term Memory

This file explains the purpose and format of `memory.md`. Each agent's actual memory is stored in its own subdirectory, alongside its `soul.md` (e.g., `mimi/memory.md`).

## Overview

The agent reads its memory file on every startup and updates it whenever the user asks to "remember" something. Memory persists across sessions.

## Directory Structure

```
<agent_name>/
  soul.md      ← personality for this agent
  memory.md    ← memory for this agent (this file)
```

## Template

Each agent's `memory.md` follows this format. Channel and skill configuration are written to their own dedicated sections:

```markdown
# Memory — <Agent Name>

## User Info

- **Name**: ...
- **Language**: ...

## User Preferences

- ...

## Channel: <channel_name>

(Configuration produced by the channel during initialization, e.g.:)
- **Chat ID**: ...

## Skill: Scheduled Tasks

(Configuration produced by the scheduled_tasks skill, e.g.:)
- **Timezone**: Asia/Shanghai
- Remind me to take medicine at 08:00
- ...

## Things to Remember

- ...
```

## Section Rules

| Section | Written by | Description |
|---|---|---|
| **User Info** | Onboarding (instruction) | Name, language, etc. |
| **User Preferences** | Agent at runtime | Anything the user asks to remember about preferences |
| **Channel: \<name\>** | Channel initialization | Each channel writes its own section with channel-specific config |
| **Skill: \<name\>** | Skill onboarding setup | Each skill writes its own section with skill-specific config and data |
| **Things to Remember** | Agent at runtime | Free-form notes the user asks the agent to remember |

## Notes

- The agent writes to `<agent_name>/memory.md` whenever the user says "remember this" or similar
- Before answering memory-related questions, the agent always re-reads this file
- Users can ask the agent to add, modify, or remove any entry
- Each channel and skill is responsible for reading/writing only its own section
