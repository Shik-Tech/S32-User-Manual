# Change safely {#change-safely-page}

The agent follows one order: read, preview, ask, change, check. It reads the state first, previews a change with `--dry-run`, shows you the preview and asks, changes only after your yes, then reads again to confirm.

## What a change prints

A change prints what it did, then the new state. Read it before you go on. `s32 rack list` reads it again, as the ground rules ask.

<!-- s32-check -->
```bash
s32 rack set 1 bypass=on
s32 rack list
```

**Output**

<!-- s32-output -->
```text
rack 1 bypass=on
  [1] id=100  bypass=on in=yes devices=1 Rack 1
```

## Commands that ask first {#change-safely}

A few commands change a lot at once, so the S32 asks first. Each needs your approval and `--yes`:

- `project delete` removes a stored project.
- `project load` replaces the live project.
- `push --activate` sends a project and plays it.
- `rack clear` removes every device in a rack.
- `rack duplicate` copies a rack over another.
- `snapshot delete` removes a snapshot.
- `globals set` changes a global setting.

Before one of them, the agent backs up the project and previews the change with `--dry-run`. It shows you the preview and asks. It adds `--yes` only after you say yes to that exact change.

<!-- s32-check -->
```bash
s32 active
s32 pull <VOLUME> <UID> s32-backup-<UID>-<DATE_TIME>.s32p
s32 project load internal 2 --dry-run
```

**Ask:** "Load project 2? Changes you have not saved in the live project are lost."

<!-- s32-check approved -->
```bash
s32 project load internal 2 --yes
s32 active
```

**Output**

<!-- s32-output -->
```text
loaded uid 2 job 1
volume=0 uid=2 dirty=0 dirtyGen=0 name=Project 2
```

## Exit codes and errors

Every command ends with an exit code. A script stops at the first one that is not 0.

| Code | Meaning | What to do |
|---|---|---|
| `0` | ok | Done. Read the new state. |
| `1` | local file/OS error | A file on this computer. Check the path. |
| `2` | usage | A typo or a missing value. Nothing was sent. Read the error. |
| `3` | link | No answer from the S32, or the s32 CLI and your S32's firmware speak different editor protocols. Do what the error's `next:` line says. If the error names the older side, a retry or another cable does not help. |
| `4` | refused by the S32 | The S32 said no, or the target is not in its list. Do not retry. If the error says the S32 changed during the read, tell the user and run the command again when they are done. An add that says it was saved is saved: do not add it again. |
| `5` | not confirmed | A command that asks first ran without `--yes`, or the answer was no. Nothing changed. |

A typo in a key name stops the command with exit code 2. The S32 is not changed. The error shows the keys the command takes, and the next line suggests the closest one.

<!-- s32-check exit=2 -->
```bash
s32 rack set 1 bypas=on
```

**Output**

<!-- s32-output stderr -->
```text
error: unknown key 'bypas'; usage: rack set bypass=on|off
did you mean: bypass=
```

## What the CLI cannot do

Some things happen on the S32 panel only: firmware update, formatting, compacting or migrating storage, pressing panel buttons, and turning ARRANGE on. The CLI and the web editor use the same USB connection, so only one app at a time can be connected.


- [Overview](./index.md) — What an agent can do with your S32, in three steps
- [Set up your agent](./setup.md) — Paste one block; the agent installs, checks and saves the rules
- [Prompts and examples](./prompts.md) — Quick starters and prompts for racks, modulation and knob mappings, with checked commands
- [For agents](./for-agents.md) — What agents read: llms.txt, Markdown copies, finding commands

