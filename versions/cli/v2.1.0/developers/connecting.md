# Connect

The s32 CLI talks to a real S32 (`--usb`) or to a simulator inside the s32 CLI process (`--loopback`).

| | `--loopback` | `--usb` |
|---|---|---|
| Target | Simulator | The S32 on USB |
| Changes last | Until the process ends | Live on the S32. `project save` keeps them |
| Device types | Every device the firmware has, with its real parameters | Every device the firmware has |

With no flag, the s32 CLI uses the S32 on USB. When it finds none it stops with an error (exit code 3); it never switches to the simulator. Add `--loopback` for the simulator.

If it does not connect, run `s32 doctor`.

## USB

Only one program can hold the editor interface. Close the web editor (any browser tab using it) first.

Holding the interface means full control: the s32 CLI can read and overwrite projects, change mappings and capture the screen. There is no password. `pull` a copy of the project you care about before you start.

These commands ask first. `--yes` skips the question. A [`--script`](./simulator.md#script) does not prompt and does not imply `--yes`, so pass `--yes` for a script that runs one of them:

| Command | Effect |
|---------|--------|
| `push --activate` | Replaces the project you are playing |
| `project delete` | Deletes a stored project |
| `project load` | Replaces the project you are playing |
| `rack clear` | Removes every device in the rack |
| `rack duplicate` | Overwrites the destination rack |
| `snapshot delete` | Deletes a stored snapshot |
| `globals set` | Writes project clock settings and S32 settings |

If a run is stopped halfway (Ctrl-C), the next run still works and ends the stopped push or pull.

`S32_DEBUG=1` prints every message the s32 CLI sends and receives.
