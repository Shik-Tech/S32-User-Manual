# s32 CLI

The **s32 CLI** drives the S32 from a terminal or a script: pull and push projects, edit macros, LFOs and racks, recall snapshots, grab the screen.

| | |
|---|---|
| CLI | `s32` 1.3.46 |
| Protocol | 4 |
| Firmware | the version in the site header |

The s32 CLI talks to the **S32 Editor** USB interface, the same one the [web editor](https://editor.shik.tech) uses. It is not USB MIDI and not the USB drive. Actions that only work on the panel (format, firmware update, text entry) stay on the panel.

Knob, macro, rack, device, LFO and snapshot numbers in commands count from 0. `macro get 4 1` is knob 05, macro 2 on the screen.

A setting is always `key=value` (`sync=1bar`, `in=no`, `bypass=on`). A word without `=` only names what the command works on: a rack, device, knob, slot, part or name. A misspelled or unknown key, a missing value or a leftover word is refused before anything reaches the S32.

## Start here

1. [Install](./getting-started.md)
2. [Connect](./connecting.md)
3. [Examples](./examples.md)
4. [Use the S32 with AI](/ai/) — set up an AI agent to work with your S32

## Commands

- [Session](./commands/session.md): `hello`, `ping`, `reset`, `job`, `schema`
- [Projects](./commands/projects.md): list, pull, push, create, load, save
- [Macros](./commands/macros.md): modes, knob names, knob maps
- [LFOs](./commands/lfos.md): settings and assignments
- [Racks and devices](./commands/racks.md)
- [Maps, meta, and globals](./commands/extmap-meta.md)
- [Snapshots](./commands/snapshots.md): banks, morph, Arrange
- [View, screen, and events](./commands/view.md)

The [USB protocol](./protocol.md) page covers the interface and session for people writing their own client.
