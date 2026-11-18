# Session

```bash
s32 --loopback hello
s32 --loopback ping
s32 --loopback schema
s32 --loopback job
s32 --loopback reset
```

| Command | What it does |
|---------|--------------|
| `hello` | Protocol and firmware versions, capability bits, `usbBudget` (what the USB-C port may draw: `DEFAULT`, `1.5A`, `3.0A`), serial number, limits (knobs, macros, racks, LFOs, snapshots, screen size) |
| `ping` | Empty round trip. The interface is alive |
| `reset` | Stops any open push or pull, clears queued events, subscribes again |
| `job [HANDLE]` | Status of a storage job (save, load, copy). No handle: the last job. `Busy` means ask again |
| `abort` | Stops an open `push`. Safe when nothing is being written |
| `schema` | Device types, parameters and enum tables (macro modes, menus, LFO parameters) |
| `version` | s32 CLI and protocol versions. In a REPL, also the firmware |
| `help [COMMAND]` | The overview, or the help of one command or group (same as `COMMAND --help`) |
| `completion bash\|zsh` | A shell completion script on stdout. No S32 needed |
| `commands` | Every command with its one-line summary. `commands --json` describes every command, key and example for programs. No S32 needed |

`s32 --version` needs no S32.

## Help

```bash
s32 --help                  # the overview
s32 macro --help            # every macro command with an example
s32 macro set --help        # arguments, keys with their allowed values, examples
s32 help macro set          # the same
```

Help comes from the same declarations that check your input, so a key listed in help is a key the command accepts. None of it needs an S32. `--help` works anywhere on a command line, in a script line and in a REPL line.

## Exit codes

| Code | Meaning |
|------|---------|
| `0` | OK |
| `1` | Local file or OS error: a script, input or output file on this computer |
| `2` | Usage: unknown command or key, a missing or malformed argument |
| `3` | Link: no reply (`timed out`), a malformed reply, USB lost, or the s32 CLI and your S32's firmware speak different editor protocols |
| `4` | Refused: the S32 answered with a status (`NotFound`, `Busy`, ...), the target does not exist, or the S32 was edited (panel or web editor) while s32 read a list |
| `5` | Not confirmed: a command that asks first ran without `--yes` and no terminal, or you answered no |

A script exits with the code of the line that failed. Every error is `error: <what>` followed by one line that says what to do next: the usage line, a close command or key (`did you mean: s32 macro set`), or a hint such as `next: check the USB cable and close the web editor, then run 's32 hello'`.

```bash
s32 macor get 4 1
# error: unknown command 'macor'
# did you mean: s32 macro
```

When the s32 CLI and your S32's firmware speak different editor protocols, the error names the older side and says how to update it. The numbers are the editor protocol of each side:

```text
error: the s32 command-line tool is older than your S32's firmware (the tool speaks editor protocol 4, your S32 speaks 5)
next: update the s32 command-line tool: run the one-line install again (https://s32-manual.shik.tech/developers/getting-started)

error: your S32's firmware is older than the s32 command-line tool (your S32 speaks editor protocol 3, the tool speaks 4)
next: update your S32's firmware: SETTINGS > FIRMWARE > FIRMWARE UPDATE (https://shik.tech/support/manuals-firmware/)
```

## Quiet

`-q` (or `--quiet`) prints results only: no USB warning, no script echo (`--- [N] line`), no `script passed`, no retry notices, no REPL banner. Errors and their next-step line always print. The state a [set prints back](#a-set-prints-the-state-it-left) is a result, so `-q` keeps it.

## JSON output

`--json` prints the result of a command as one JSON object on one line, for your own tools and scripts. It works on every read command and on every set (see below):

```bash
s32 --loopback status --json
```

```json
{"bpm":120.00,"bpmX100":12000,"clockMode":"internal","externalClock":"off","transport":"stopped","outputFull":"off"}
```

- **One object, one line.** stdout carries only that object: no USB warning, no echo, no banner. `--json` implies `-q`.
- **Keys are the names you type.** A field that a set command takes as `key=value` has that key as its name (`channel`, `cvOut`, `targetLfo`, `cc`). When the text prints another spelling the key is the accepted one: `extmap list` prints `num=` and `dev=`, the JSON says `cc` and `device`. A field no command takes keeps the word the text prints (`jobState`, `backStack`).
- **Words, numbers, null.** An enum or yes/no is the exact word you can pass back (`"on"`, `"global"`, `"yes"`). A number is a JSON number with the text's precision (`0.500`, `120.00`). Indices start at 0. A name is a string with no padding. An empty slot, a rack or device that is not set, and a field that does not apply to the target are `null`. Masks stay hex strings (`"0x9000"`). A few numbers keep the value the text prints for "none": `sub` 255, `active` 255 in a snapshot list.
- **Fields the text prints only sometimes** are always there, `null` when the text leaves them out: `job --json` has `volume` (`0` internal, `1` card), set only for a save-as or duplicate job.
- **Lists** are a plural array plus their counters (`{"count":1,"max":8,"racks":[...]}`); every row carries its `index` or `slot`.
- **Escaping.** Strings follow RFC 8259: `"` and `\` are escaped, control bytes become `\u00XX`, valid UTF-8 passes through, and a byte that is not valid UTF-8 becomes `\u00XX` (its Latin-1 reading), so the output is always valid JSON.
- **`schema --json`** is `schema --emit-schema-json`, unchanged. `layout` keeps `--emit-layout-json`.

A command with no JSON form (`watch`, `pull`, `screen png`, `pause`, `help`, `completion`, `layout`, and every write that does not print a state back, such as `device add` or `project delete`) refuses `--json` with exit code 2 and sends nothing:

```text
error: --json is not available for 's32 watch'
```

### Errors

With `--json` a failure prints exactly one JSON object on stderr and leaves stdout empty. The exit code is the same as without `--json`.

```bash
s32 --loopback macro set 4 1 chanel=3 --json
```

```json
{"error":{"code":2,"class":"usage","message":"unknown key 'chanel'; usage: macro set …","next":"did you mean: channel=","suggestions":["channel="],"status":null,"applied":false}}
```

| Field | Meaning |
|-------|---------|
| `code` | The [exit code](#exit-codes) |
| `class` | One word per code: `local` (1), `usage` (2), `link` (3), `refused` (4), `declined` (5) |
| `message` | The text after `error: `, unchanged |
| `next` | The next-step line, or `null` when there is none |
| `suggestions` | The "did you mean" candidates, in the order the text prints them, or `[]` |
| `status` | The S32 status word (`"NotFound"`) for class `refused`; `"VersionMismatch"` when the tool and the S32 speak different editor protocols (class `link`); otherwise `null`. It is also `null` for the refused error "the S32 changed while s32 read the list", where the S32 sent no status |
| `applied` | `true` when the S32 acknowledged a write of this command before it failed, so a program can tell "nothing changed" from "changed, not confirmed". For class `link` a write whose reply was lost may still have landed on the S32. It is `true` for "added assignment N, but the S32 changed while s32 read …": the write was saved before the read-back saw the change |

Errors that come before the command is known are JSON too: an unknown command, an unknown word under a group (`macro sett 4 1`, with the close command in `suggestions`). In a script or REPL line the object also has `line` (the line number) and `text` (the line).

`--json` does not prompt, not even on a terminal. A command that asks first and has a JSON form (`globals set`) exits with code 5 and the `declined` error unless you pass `--yes`; nothing is sent. The other commands that ask (`rack clear`, `project delete`, ...) have no JSON form, so `--json` on them is the usage error above, with or without `--yes`.

### A set prints the state it left

After a set succeeds, the CLI reads the state back from the S32 and prints it: your own confirmation line first (unchanged), then the lines the matching `get` prints. The state comes from a real request, so it shows what the S32 stored, which can differ from what you sent (a value it rounded or limited).

```text
$ s32 --loopback macro set 4 1 channel=3
macro 4/1 updated
pot 4 macro 1: mode=disabled msb=0 lsb=0 channel=3 min=0.000 max=127.000 ...
```

- **Lists** print only the row you changed, with no list header: `rack set 0 bypass=on` prints `rack 0 bypass=on` and then `  [0] id=100  bypass=on in=yes devices=1 Rack 1`.
- **`--json`** prints only the read-back object (for a list, the changed row's object); the confirmation line is not printed.
- **`device param set`** reads back one parameter per `PARAM=VALUE`, in the order given: one line each in text, one object with a `params` array in JSON.
- **Five sets already printed their state.** `snapshot parent set`, `snapshot flags set`, `snapshot bank set`, `device host-device ... host=` and `extmap set` print the state line themselves, so the same line shows twice: the confirmation, then the read-back.
- A read-back that fails never repeats the set. Text: the confirmation line, then the read's `error:` line. `--json`: one error with `"applied":true`.

The set commands with a read-back are `macro set`, `potname set`, `lfo cfg set`, `lfo assign add|set`, `rack set|rename`, `device param set`, `device host-device`, `stepseq set`, `extmap add|set`, `meta set`, `globals set` and `snapshot sequence|arrange|flags|parent|bank|morph set`.

### Scripts and the REPL

`s32 --json --script FILE` and `s32 --json repl` print one JSON object per command line that succeeds, and nothing else: no `--- [N]` echo, no `script passed:`, no REPL banner, no `s32> ` prompt, no retry notices (a retried line reports only its final result). A failing script line prints one JSON error with `line` and `text`, and the script stops with that line's exit code. A failing REPL line prints the same error and the REPL goes on. Put `--json` on a single line of a normal script to get JSON for that line only; the echo and the `script passed:` line stay.

### Stability

Within a CLI major version (`1.x.y`), a JSON key or error field is never renamed or removed and its type never changes. New keys can be added: ignore keys you do not know. Every read command has a checked-in JSON Schema in `tools/s32-cli/json/`.

## Preview before a change

`--dry-run` works on every command that changes the S32. It checks the command exactly as the real run does, reads what it needs from the S32, prints the change it would make and exits 0. **It sends no write**, and it needs no `--yes` and does not prompt, even for `project delete`, `project load`, `push --activate`, `rack clear`, `rack duplicate`, `snapshot delete` and `globals set`.

```bash
s32 --loopback macro set 4 1 channel=3 msb=74 --dry-run
```

```text
dry-run: s32 macro set 4 1 channel=3 msb=74 (nothing sent)
  channel: global -> 3
  msb: 0 -> 74
```

- A key that keeps its value prints `msb: 0 (unchanged)`. A key that creates a row (`extmap add`, `lfo assign add`) prints `ch: (new) -> 1`.
- A command that changes no named key prints one `  would …` sentence: `would permanently remove the stored project vol=0 uid=3 name=Project 3`, `would remove every device in rack 0 (id 100, 1 devices)`.
- Wrong arguments, an unknown key or a project that does not exist fail with the same exit code and message as the real command. `--dry-run` on a command that does not change the S32 (`macro get`, `commands`) is a usage error.
- With `--json` it prints one object: `{"dryRun":true,"command":"macro set","target":{"pot":4,"macro":1},"destructive":false,"changes":[{"key":"channel","from":"global","to":"3"}],"would":null}`. `from` and `to` are the words the CLI prints and accepts; `from` is `null` for a new row. `target` has one member per target argument you typed (a number when you typed a plain integer, otherwise a string).
- `--dry-run` checks only what the CLI can check: the S32 can still refuse the real write. Its reads count against the S32's rate limit like any read.

## Completion

```bash
source <(s32 completion bash)      # bash: add this line to ~/.bashrc
source <(s32 completion zsh)       # zsh: add this line to ~/.zshrc
```

Press Tab to complete commands, verbs, `key=` names and the words after `key=` (`macro set 4 1 mode=` lists the macro modes).

## Schema JSON

```bash
s32 --usb schema --emit-schema-json schema.json
```

Over `--loopback` the schema lists every real device type with its real parameters ([simulator](../simulator.md)).
