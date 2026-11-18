# Maps, meta, globals

```text
s32 --loopback repl
s32> extmap add ch=1 cc=20
s32> extmap list
s32> extmap set 0 ch=2 cc=21
s32> extmap remove 0
s32> meta get
s32> meta set bpm=137.5 description=Live-set-A
s32> globals get
s32> globals set bpm=128 clockMode=internal
```

## External MIDI map

Incoming CC on a channel, mapped to a device parameter, an LFO parameter, a rack
BYPASS or a snapshot control.

| Command | What it does |
|---------|--------------|
| `extmap list` | Every row, with its target |
| `extmap add ch=CH cc=CC` | Adds a row for that channel (1–16) and CC |
| `extmap set INDEX ch=CH cc=CC` | Rewrites that row in place |
| `extmap remove INDEX` | Deletes that row |

Target words for `extmap add` and `extmap set`:

| Target | Fields | Example |
|--------|--------|---------|
| `target=device` (default) | `rack=`, `device=`, `param=`, optional `sub=` for a Step Seq step | `extmap add ch=1 cc=20 target=device rack=100 device=1000 param=cutoff sub=1` |
| `target=lfo` | `lfo=`, `domain=s32`, `lfo-param=` (also `cvParam=`) | `extmap add ch=1 cc=21 target=lfo lfo=1 domain=s32 lfo-param=rate` |
| `target=rack-bypass` (also `rack`) | `rack=` | `extmap add ch=1 cc=22 target=rack-bypass rack=100` |
| `target=snapshot-slot` | `part=`, optional `pad=` (`1`–`32`) | `extmap add ch=1 cc=23 target=snapshot-slot part=knobs pad=5` |
| `target=morph-amount` | `part=` | `extmap add ch=1 cc=24 target=morph-amount part=knobs` |

`extmap list` prints one line per row, for example:

```text
  [0] src=0 ch=1 num=20 target=device rack=100 dev=1000 param=3 sub=255
  [1] src=0 ch=1 num=21 target=lfo domain=s32 lfo=1 lfo-param=rate
  [2] src=0 ch=1 num=22 target=rack-bypass rack=100
```

A parameter has one mapping: one knob macro or one external row. Adding or
retargeting a row on a mapped parameter clears its knob mapping, the same as
picking **EXTERNAL MAPPING** on the S32. Only a change of target clears; editing
the channel or CC number of the same target keeps the knob mapping. A read of
the row shows the row that now holds the mapping.

`part=` makes the CC pick a snapshot or drive a morph. On a snapshot slot,
`pad=` maps that one pad instead of the whole bank; without it the note or
Program Change numbers the whole bank from the learned number. See
[Snapshots](./snapshots.md#mapping).

## Project meta

`meta get` prints the project's BPM, clock mode, channel, encoder button action, tags and description. `meta set bpm=NUMBER description=TEXT` writes the BPM and description and keeps the tags.

## Globals

| Key | Values | Where it lives |
|-----|--------|----------------|
| `clockMode` | `internal`, `external`, `auto` | Project |
| `bpm` | Internal clock BPM | Project |
| `globalChannel` | `1`–`16` | Project |
| `encoderBtn` | `clock`, `racks`, `lfos` | Project |
| `transport` | `stopped`, `running` | Live only |

Project values change the project you are playing. **SAVE PROJECT** (or `project save`) keeps them.

## Output after a set

`extmap add` and `extmap set` print the row they wrote (`extmap list` format, no `external mappings: N` header), `meta set` and `globals set` print the project settings ([A set prints the state it left](./session.md#a-set-prints-the-state-it-left)):

```text
$ s32 --loopback extmap add ch=1 cc=20
added external mapping row 0 (1 total)
  [0] src=0 ch=1 num=20 target=device rack=- dev=- param=255 sub=255
$ s32 --loopback --yes globals set bpm=130
globals updated
globalChannel=1 clockMode=internal bpm=130.00 transport=stopped encoderBtn=clock octaveEnc=0
```

`extmap set` already printed the row it sent, so the row shows twice: what the CLI sent, then what the S32 stored (they differ only when the S32 changes the row).

With `--json`, `extmap list` names the fields as `extmap add` takes them: `num=` is `cc`, `dev=` is `device`, `lfo-param` stays `lfo-param`, a field that does not apply to the row's target is `null` (`rack` is `null` while the rack is unset). `globals set --json` needs `--yes`: under `--json` it does not prompt, so without `--yes` it exits `5` with the `declined` error.
