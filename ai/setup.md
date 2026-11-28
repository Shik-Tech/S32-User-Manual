# Set up your agent

Paste one block into your agent. It installs the s32 CLI, checks it, and saves the s32-cli skill with the rules to follow. You only paste: the words in the block are for the agent. Numbers match the panel: knob 01 is `1`, rack 1 is `1`.

## Before you start

Close the [web editor](https://editor.shik.tech). The S32 talks to one app at a time. That is all you do before you paste. If you do not have the s32 CLI yet, the block below installs it for you, or use the [one-line install](../developers/getting-started.md). Update your S32 to the latest firmware first: [Update the firmware](../settings/#update-firmware).

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

2. Check the s32 CLI: run `s32 --version`. It must be 2.1.0 or later. If it is older, run the install of step 1 again.

3. Check it with the S32. I close the S32 web editor first: the S32 talks to one app at a time. Run `s32 --usb hello`. It works when it prints my S32's serial number and firmware version.

4. If step 3 fails, run `s32 doctor` and show me what it prints. If it says no S32 was found, USB is not available or access was denied, while my S32 is plugged in and the web editor is closed, your sandbox may be blocking USB. Then stop, tell me so, and ask me for permission to run s32 outside your sandbox. Ask for s32 only, and change no other setting. Once I allow it, run `s32 doctor` again, then `s32 --usb hello`. If it still fails, do what its `next:` line says and tell me. Never check the connection with --loopback: the simulator always answers, and that says nothing about my S32.

5. Save the skill: the lines after the line "=== s32-cli skill (SKILL.md) ===" are the text of the s32-cli skill, with the rules to follow. Leave that marker line itself out. If you support Agent Skills, save those lines as a new skill named s32-cli (they are the skill's SKILL.md). Otherwise save the same lines to the rules file you read at the start of every session: AGENTS.md, CLAUDE.md or your tool's own rules file. Tell me where you saved it.

6. Check the setup: run `s32 --version` and `s32 --usb active`, then tell me the s32 CLI version, the name of the loaded project, and where you saved the skill. From now on, follow the skill. For anything it does not cover, read https://s32-manual.shik.tech/llms.txt.

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
- Run `s32 --usb hello`. If it fails, run `s32 doctor`, show me what it prints and stop. If it says no S32 was found, USB is not available or access was denied while the S32 is plugged in, your sandbox may be blocking USB: ask me for permission to run s32 outside the sandbox, for s32 only. If `s32 doctor` is an unknown command, the s32 CLI is too old: ask me, then run the one-line install again.
- From s32 2.1.0 on, every s32 command without `--loopback` talks to the S32 on USB and never switches to the simulator on its own. With an older s32 CLI (`s32 --version`), always pass `--usb` to commands for the S32. Use the simulator only when I ask to rehearse: add `--loopback`, for example `s32 --loopback hello`, and tell me you are using it. Its success says nothing about the S32. It starts fresh on every run, and its ids, names and ranges are not the S32's.
- Once per session, after `s32 --usb hello` works, check the S32's firmware. Take the version from its `firmwareVersion` line and the latest published version from the `version` field of https://github.com/Shik-Tech/public-resources/raw/refs/heads/main/S32-Firmware/latest.json, or the version listed on https://shik.tech/support/manuals-firmware/. Compare them as three numbers, x.y.z: first x, then y, then z (1.1.10 is newer than 1.1.9). If the S32's version is older, tell me: "A newer S32 firmware is available (<latest>); your S32 has <S32 version>. Update it on the panel: https://s32-manual.shik.tech/settings/#update-firmware". Only the panel updates the firmware: never update it yourself and never suggest an s32 command for it. If the versions are equal or the S32's is newer, say nothing about it. If you cannot fetch the version, or one of them is not three numbers, skip this check and say nothing about it.
- Numbers match the panel: knob 01 is `1`, macro 1 is `1`, rack 1 is `1`, and a device's place in `device list` is its number (MIDI IN is `1`). Rack and device ids (`id=` in `rack list` and `device list`) are not places. Ids are different on every S32: read them with `s32 rack list` and `s32 device list 1` just before you use one, after your last change to that rack. Never copy an id from an example or from the simulator.

## Ground rules

- Every setting is `key=value`. Plain numbers and names only choose what you change. Numbers match the panel: knob 01 is 1, rack 1 is 1.
- Read before you change, and read again after.
- Before anything that asks first, back up the active project: run `s32 active`; if it shows `dirty=1`, tell me the backup misses unsaved changes and ask whether to save first with `s32 project save --wait`. Then run `s32 pull <VOLUME> <UID> s32-backup-<UID>-<DATE_TIME>.s32p` with its volume and uid and the current date and time. If s32 pull says the file already exists, choose a new name. Never pass --force.
- Preview a change with `--dry-run` and show me the preview. Pass `--yes` only after I approved that exact change.
- Stop on any exit code other than 0 and show me the error.
- Never run a command that s32 commands --json marks "service": true, and never pass --dev. If one seems needed, stop and tell me.

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
- Names work for `rate`, `gate` and `port`. Every other parameter is its number: `device param set 1 2 87=4`.
- `device add` puts a new device before MIDI OUT. If the rack already has a Clock or a MIDI OUT, do not add another.
- A new MIDI IN or MIDI OUT uses PORT USB. `port=1` is MIDI1, `port=2` is MIDI2 (MIDI OUT only: a MIDI IN stores it as MIDI1), `port=3` is USB HOST.
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
- 3: the S32 could not be reached. The error says "no S32 found on USB", "USB is not available to this program", "access denied", "another program is using the S32", "has no editor interface", "could not open the S32", "this build has no USB backend", "timed out", "HELLO failed", or that the s32 command-line tool or your S32's firmware "is older" or that they "speak different editor protocols". Do what the error's `next:` line says; `s32 doctor` shows which step fails. If the error names the older side, a retry or another cable does not help.
- 4: the S32 said no, or the target is not in its list. Read the error. Do not retry.
  If the error says the S32 changed during the read, tell the user and run the command again when they are done. An add that says it was saved is saved: do not add it again.
- 5: a command that asks first ran without `--yes`, or the answer was no. Nothing changed.

## What only the panel does

- Formatting, compacting or migrating storage.
- Firmware update. If an error says the S32's firmware is older, tell the user to update it on the panel: https://s32-manual.shik.tech/settings/#update-firmware. A retry does not help.
- Pressing panel buttons and text entry.
- Turning ARRANGE on: the CLI writes the song, the panel starts the playhead.
- The CLI and the web editor never run at the same time.

## Reference

- https://s32-manual.shik.tech/llms.txt: an index of the AI and s32 CLI pages. Every page it links, except the manual home, is a Markdown copy.
- https://s32-manual.shik.tech/ai/prompts: prompts with checked commands and read-backs.
- `s32 <command> --help` and `s32 commands --json`: every command with its keys and values.
````

To install the s32 CLI yourself, see [Manual install](../developers/getting-started.md#manual-install).

## If the agent cannot find the S32 {#sandbox}

Many agents run commands in a sandbox that can block USB. The s32 CLI then stops with `no S32 found on USB`, `USB is not available to this program` or `… (access denied)`, although the S32 is plugged in and the web editor is closed. `s32 doctor` shows which step fails. The prompt tells the agent to stop and ask you. Allow the s32 CLI to run outside the sandbox, and only the s32 CLI:

- **Codex:** approve the request when the agent asks to run the s32 CLI outside the sandbox, with the narrowest scope it offers ([Codex sandboxing](https://learn.chatgpt.com/docs/sandboxing)).
- **Claude Code:** with the sandbox on, approve the retry outside the sandbox. To allow it every time, add `"excludedCommands": ["s32 *"]` under `sandbox` in your settings. That is a standing exemption for every matching s32 CLI call, and it is your choice. A call that starts with `cd`, writes to a file or uses `$(...)` still runs in the sandbox ([Claude Code sandboxing](https://code.claude.com/docs/en/sandboxing)).
- **Other agents:** approve running the s32 CLI outside the sandbox, or add it to the commands the agent may run outside it.

If `s32 doctor` says another program is using the S32, close the web editor and any other s32 CLI session.

## Rehearse without the S32

To rehearse without hardware, ask the agent to use the simulator (`--loopback`); it tells you when it does. The simulator starts fresh on every run. Its ids, names and ranges are not your S32's, and a command that works there says nothing about your S32.

<!-- s32-skip: interactive -->
```bash
s32 --loopback repl
```


- [Overview](./index.md) — What an agent can do with your S32, in three steps
- [Prompts and examples](./prompts.md) — Quick starters and prompts for racks, modulation and knob mappings, with checked commands
- [Change safely](./safe-changes.md) — Read, preview, ask, change, check
- [For agents](./for-agents.md) — What agents read: llms.txt, Markdown copies, finding commands

