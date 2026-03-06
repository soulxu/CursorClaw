# Skill: Scheduled Tasks

Automatically execute recurring tasks at specified times each day.

## Required Allowlist Commands

None beyond the core commands (uses `Shell(sleep)` and `Shell(curl)` which are already in core).

## How It Works

On each polling cycle, check the current time in the user's timezone and compare against scheduled tasks stored in `<AGENT_DIR>/memory.md`.

### Getting Current Time

```bash
TZ=<USER_TIMEZONE> date '+%H:%M'
```

### Matching Rules

- A task matches if current time is within ±5 minutes of the target time (e.g., target 08:00 matches 07:55–08:05)
- Each task executes **at most once per day**
- Maintain an internal record of executed tasks to prevent repeats
- Reset the record when the date changes

### Task Storage

Scheduled tasks are stored in `<AGENT_DIR>/memory.md` under the **"Skill: Scheduled Tasks"** section. Users can add, modify, or remove tasks by telling the agent (e.g., "remind me to X at Y every day", "cancel the 8am reminder").

### Example Task Execution

```
Polling loop:
  1. sleep 5
  2. Check time: TZ=America/New_York date '+%H:%M' → 08:02
  3. memory.md has: "Remind user to take medicine at 08:00"
  4. 08:02 is within ±5 min of 08:00, and not yet executed today
  5. → send via channel: "Good morning! Time to take your medicine 💊"
  6. Mark "take medicine" as executed for today
  7. Continue to read watch output and process messages
```

### External Data Tasks

When tasks include fetching external data (weather, stocks, etc.), use curl to query public APIs:

- Weather: `curl -s "wttr.in/<City>?format=..."`
- Stocks: any public stock API

## Onboarding Setup

During onboarding, perform these steps:

1. **Detect timezone**: run `readlink /etc/localtime | sed 's|.*/zoneinfo/||'` to get the IANA timezone (e.g., `Asia/Shanghai`). Save it to the **"Skill: Scheduled Tasks"** section in `<AGENT_DIR>/memory.md`. If detection fails, ask the user where they live and derive the timezone.
2. **Ask the user** (via the channel):
   > "Would you like any daily scheduled reminders? For example: 'Remind me to take medicine at 8am', 'Send weather report at 7am'. You can always add more later. Type 'skip' if not needed."
