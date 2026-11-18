# Set up your agent

Paste one block into your agent. It installs the s32 CLI, checks it, and saves the s32-cli skill with the rules to follow. You only paste: the words in the block are for the agent. Numbers count from 0: knob 01 is `0`, rack 1 is `0`.

## Before you start

Close the [web editor](https://editor.shik.tech). The S32 talks to one app at a time. That is all you do before you paste. If you do not have the s32 CLI yet, the block below installs it for you, or use the [one-line install](../developers/getting-started.md).

## Paste this into your agent {#starter-prompt}

The block works for every agent. Use the Copy button in its corner, then paste it at the start of a session. It holds the setup steps and the s32-cli skill. The agent asks before it installs anything.

<!-- s32-prompt -->
````text
Set up the s32 CLI so you can work with my S32 MIDI controller. The s32 CLI talks to the S32 over USB. Do the steps in order and tell me how each one went.

1. Install the s32 CLI. Show me the command and wait for my yes before you run it.
   macOS or Linux: curl --proto '=https' --tlsv1.2 -fsSL https://github.com/Shik-Tech/S32-CLI/releases/latest/download/install.sh | sh
   Windows (PowerShell): irm https://github.com/Shik-Tech/S32-CLI/releases/latest/download/install.ps1 | iex
   Never run sudo yourself. On Linux the installer may stop and name a command for the USB library, or print the commands that add the S32 USB rule. Both start with sudo. Show them to me and wait until I say I ran them. After the USB library, run the installer again.
   If the s32 command is not found afterwards, open a new terminal, or add the folder the installer names to PATH.

2. Check it without the S32: run `s32 --version`, then `s32 --loopback hello`. --loopback uses a simulator. It starts fresh on every run, and its ids, names and ranges are not my S32's.

3. Check it with the S32. I close the S32 web editor first: the S32 talks to one app at a time. Run `s32 hello`. If it fails, stop and show me the error.

4. Save the skill: the lines after the line "=== s32-cli skill (SKILL.md) ===" are the text of the s32-cli skill, with the rules to follow. Leave that marker line itself out. If you support Agent Skills, save those lines as a new skill named s32-cli (they are the skill's SKILL.md). Otherwise save the same lines to the rules file you read at the start of every session: AGENTS.md, CLAUDE.md or your tool's own rules file. Tell me where you saved it.

5. Check the setup: run `s32 --version` and `s32 active`, then tell me the s32 CLI version, the name of the loaded project, and where you saved the skill. From now on, follow the skill. For anything it does not cover, read https://s32-manual.shik.tech/llms.txt.

=== s32-cli skill (SKILL.md) ===
---
name: s32-cli
description: Controls the S32 MIDI controller from the command line with the s32 CLI. Builds rack device chains, maps knobs, LFOs and envelopes to device parameters and MIDI, and reads and changes macros, racks, snapshots and projects safely (read, preview, ask, change, check). Use when the user mentions the S32 or the s32 CLI, or wants to change S32 racks, devices, knobs, macros, LFOs, snapshots or projects.
---
# s32 CLI

The s32 CLI is the command-line tool for the S32 MIDI controller. It talks to the S32 over USB. Use it to read and change projects, racks and devices, knob macros, LFOs and snapshots.

## Before you start

- If the s32 CLI is not installed, ask me, then run the one-line install for my system from https://s32-manual.shik.tech/developers/getting-started. On Windows run it as `powershell -NoProfile -Command 'irm https://github.com/Shik-Tech/S32-CLI/releases/latest/download/install.ps1 | iex; exit $LASTEXITCODE'` (single quotes, so your shell does not expand `$LASTEXITCODE` first), so a failed install returns a non-zero code.
- Close the S32 web editor. The S32 talks to one app at a time.
- Run `s32 hello`. Stop if it fails.
- To try a command without touching the S32, add `--loopback`, for example `s32 --loopback hello`. The simulator starts fresh on every run. Its ids, names and ranges are not the S32's.
- Numbers count from 0: knob 01 is `0`, macro 1 is `0`, rack 1 is `0`, and a device's position in `device list` is its number (MIDI IN is `0`). Rack and device ids (`id=` in `rack list` and `device list`) are not positions. Ids are different on every S32: read them with `s32 rack list` and `s32 device list 0` just before you use one, after your last change to that rack. Never copy an id from an example or from the simulator.

## Ground rules

- Every setting is `key=value`. Plain numbers and names only choose what you change. Numbers count from 0: knob 01 is 0, rack 1 is 0.
- Read before you change, and read again after.
- Before anything that asks first, back up the active project: run `s32 active`; if it shows `dirty=1`, tell me the backup misses unsaved changes and ask whether to save first with `s32 project save --wait`. Then run `s32 pull <VOLUME> <UID> s32-backup-<UID>-<DATE_TIME>.s32p` with its volume and uid and the current date and time. Never replace an existing file.
- Preview a change with `--dry-run` and show me the preview. Pass `--yes` only after I approved that exact change.
- Stop on any exit code other than 0 and show me the error.

## Workflow

1. Discover: `s32 --help` lists the commands, `s32 <command> --help` lists the keys of one command, `s32 commands --json` lists everything for a program.
2. Read the current state: `rack list`, `device list`, `knobmap get`, `lfo cfg get`, `active`.
3. Preview the change with `--dry-run` and show it to the person.
4. Ask. Wait for a clear yes for that exact change.
5. Change. A change prints what it did, then the new state.
6. Read it back and compare with what was asked.
7. Report: the commands you ran, what changed (old -> new), and anything the S32 refused.

## Device parameters

- `s32 schema --emit-schema-json` lists every device with each parameter's id, name and range.
- Names work for `rate`, `gate` and `port`. Every other parameter is its number: `device param set 0 1 87=4`.
- `device add` puts a new device before MIDI OUT. If the rack already has a Clock or a MIDI OUT, do not add another.
- A new MIDI IN or MIDI OUT uses PORT USB. `port=1` is MIDI1, `port=2` is MIDI2, `port=3` is USB HOST.
- On SYNCED, times are steps of 24 per quarter note: 96 = 1/1, 48 = 1/2, 24 = 1/4, 12 = 1/8, 6 = 1/16.

## Commands that ask first

Each needs the person's approval and `--yes`. Preview it with `--dry-run` first.

- `project delete` removes a stored project.
- `project load` replaces the live project.
- `push --activate` sends a project and plays it.
- `rack clear` removes every device in a rack.
- `rack duplicate` copies a rack over another.
- `snapshot delete` removes a snapshot.
- `globals set` changes a global setting.

## Exit codes

```text
0 ok · 1 local file/OS error · 2 usage · 3 link
4 refused by the S32 · 5 not confirmed
```

- 0: done. Read the new state.
- 1: a file on this computer. Check the path.
- 2: a typo or a missing value. Nothing was sent. Read the error and `s32 <command> --help`.
- 3: no answer from the S32, or the s32 CLI and your S32's firmware speak different editor protocols. Do what the error's `next:` line says. If the error names the older side, a retry or another cable does not help.
- 4: the S32 said no, or the target is not in its list. Read the error. Do not retry.
  If the error says the S32 changed during the read, tell the user and run the command again when they are done. An add that says it was saved is saved: do not add it again.
- 5: a command that asks first ran without `--yes`, or the answer was no. Nothing changed.

## What only the panel does

- Firmware update, formatting, compacting or migrating storage.
- Pressing panel buttons and text entry.
- Turning ARRANGE on: the CLI writes the song, the panel starts the playhead.
- The CLI and the web editor never run at the same time.

## Reference

- https://s32-manual.shik.tech/llms.txt: an index of the AI and s32 CLI pages. Every page it links, except the manual home, is a Markdown copy.
- https://s32-manual.shik.tech/ai/prompts: prompts with checked commands and read-backs.
- `s32 <command> --help` and `s32 commands --json`: every command with its keys and values.
````

To install the s32 CLI yourself, see [Manual install](../developers/getting-started.md#manual-install).

## Rehearse without the S32

To rehearse without hardware, have the agent work against the simulator. The simulator starts fresh on every run. Its ids, names and ranges are not your S32's.

<!-- s32-skip: interactive -->
```bash
s32 --loopback repl
```


- [Overview](./index.md) — What an agent can do with your S32, in three steps
- [Prompts and examples](./prompts.md) — Quick starters and prompts for racks, modulation and knob mappings, with checked commands
- [Change safely](./safe-changes.md) — Read, preview, ask, change, check
- [For agents](./for-agents.md) — What agents read: llms.txt, Markdown copies, finding commands

