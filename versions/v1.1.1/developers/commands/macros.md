# Macros and knobs

Each of the 32 knobs has four [macros](/macros/). A macro turns knob motion into MIDI, a rack device parameter or an S32 LFO parameter.

```bash
s32 --loopback macro get 4 1
s32 --loopback macro set 4 1 mode=cc msb=74 channel=3
s32 --loopback potname get 4
s32 --loopback potname set 4 Cutoff
s32 --loopback knobmap get 0
s32 --loopback knobmap copy 3 9
```

## Modes

| `mode=` | Also | Sends | Page |
|---------|------|-------|------|
| `disabled` | `off` | Nothing | [Disabled](/macros/disabled) |
| `cc` | `control-change` | Control Change | [Control Change](/macros/control-change) |
| `pc` | `program-change` | Program Change | [Program Change](/macros/program-change) |
| `cc14` | `fourteen-bit` | 14-bit Control Change | [14-bit CC](/macros/fourteen-bit-cc) |
| `aftertouch` | `at` | Channel aftertouch | [Channel aftertouch](/macros/channel-aftertouch) |
| `poly-aftertouch` | `poly-at`, `pat` | Poly aftertouch (`msb` is the note) | [Poly aftertouch](/macros/poly-aftertouch) |
| `nrpn` | | NRPN | [NRPN](/macros/nrpn) |
| `rpn` | | RPN | [RPN](/macros/rpn) |
| `pitch-bend` | `pb` | Pitch bend | [Pitch bend](/macros/pitch-bend) |
| `sysex` | | SysEx | [SysEx](/macros/sysex) |
| `device` | `device-parameter` | A rack device parameter | [Device parameter](/macros/device-parameter) |
| `rack` | `rack-parameter` | A rack's BYPASS | [Rack Control](/macros/rack-control) |
| `s32-lfo` | `lfo` | An S32 LFO parameter | [S32 LFO](/macros/s32-lfo-parameter) |

## `macro get` / `macro set`

`macro get KNOB MACRO`: knob `0`–`31`, macro `0`–`3`.

`macro set KNOB MACRO key=value …` changes only the keys you name:

| Key | Values |
|-----|--------|
| `mode` | A name from the table above |
| `msb` | CC number, program number, NRPN/RPN MSB, note, … |
| `lsb` | LSB, where the mode has one |
| `channel` | `global` or `1`–`16` |
| `min` / `max` | Range. 7-bit modes: `0`–`127`. `cc14`, `nrpn`, `rpn`, `pitch-bend`, `sysex`: `0`–`16383`. `mode=device`: the parameter's range, except a time that changes with SYNC (see below) |
| `invert` | `on` / `off` |
| `outputs` | `none`, `usb`, `out1`, `out2`, or a `+` mix such as `usb+out1` |
| `pickup` | `on` / `off` |
| `in` | IN SNAPSHOT, `yes` / `no` |
| `rack` / `device` / `param` | `mode=device`: rack and device ids from `rack list` / `device list`, and a parameter name (`bypass`, `rate`, `gate`, …) or number from `schema`. `mode=rack`: the rack id from `rack list`; `param` is `bypass` |
| `sub` | `mode=device` on a Step Seq step: the step, `0`–`15` |
| `lfo` / `cvParam` | `mode=s32-lfo`: the LFO (from `0`) and its setting (`enabled`, `wave`, `rate`, `sync`, `phase`, `depth`, `offset`) |
| `name` | The label for a named control |

On a time that changes with SYNC (Delay `delay-time`, Arpeggiator `rate`, Envelope `attack` / `hold` / `decay` / `release`), `min` and `max` are positions from `0` to `1` of the range it has in its current mode: `min=0 max=1` is the whole range on SYNCED and on FREE, and `min=0.5 max=1` stays the upper half when SYNC changes. `macro get` prints the positions. A value above `1` is read in the current unit (ticks or ms) and stored as its position.

```text
macro set 4 1 mode=cc msb=74 channel=3 outputs=usb+out1
macro set 0 2 mode=cc14 msb=1 lsb=33 channel=3
macro set 0 0 mode=device rack=100 device=1000 param=bypass
macro set 2 0 mode=s32-lfo lfo=0 cvParam=depth
macro set 3 0 mode=disabled
```

Switching into `cc14`, `nrpn`, `rpn`, `pitch-bend` or `sysex` without `min=`/`max=` sets the full 0–16383, as on the panel. For any other mode change, name `min=` and `max=` on the same line.

`target=snapshot-slot` and `target=morph-amount` with `part=` turn a macro into a snapshot picker or a morph knob. See [Snapshots](./snapshots.md#mapping).

`target=rack-bypass rack=ID` maps a macro to a rack's BYPASS, the same target
the panel's rack Options use:

```text
macro set 3 0 target=rack-bypass rack=100
```

A parameter has one mapping: one knob macro or one external MIDI row. Giving a
knob or external mapping a **new** target clears the parameter's other mapping,
as on the S32. Editing min/max, the name or the CC number of the same target
keeps the other mapping; a rejected command changes nothing. A whole knob map
(`knobmap copy`, an editor `KNOBMAP_SET`) follows the same rule, so the last
slot that writes a parameter wins.

## Knob names

`potname get KNOB` and `potname set KNOB NAME`.

## Knob map

`knobmap get [START]` lists knob names and their macros from `START`.

`knobmap copy SRC DST` copies one knob's name and all four macros onto another.
Because a parameter has one mapping, mapped parameters **move** to the
destination knob: the source knob's slots become disabled and the destination
knob's slots map them.

## Output after a set

`macro set` and `potname set` print their confirmation line, then the state the S32 now holds ([A set prints the state it left](./session.md#a-set-prints-the-state-it-left)):

```text
$ s32 --loopback potname set 4 Cutoff
pot 4 renamed
pot 4 name=Cutoff
$ s32 --loopback macro set 4 1 channel=3
macro 4/1 updated
pot 4 macro 1: mode=disabled msb=0 lsb=0 channel=3 min=0.000 max=127.000 invert=off outputs=none pickup=off in=yes rack=0 device=0 param=0 sub=255 cvOut=0 lfo=0 name=
```

With `--json`, `macro get` and `macro set` print the same object, keyed by the names `macro set` takes (`mode`, `msb`, `channel`, `outputs`, `in`, `cvOut`, ...). The shorthand keys `target`, `part` and `cvParam` have no field of their own: the object shows what they resolved to (`mode`, `rack`, `param`, `min`, `max`). `knobmap get --json` lists the knobs with their first macro.

```bash
s32 --loopback macro get 4 1 --json
s32 --loopback macro set 4 1 channel=3 msb=74 --dry-run
```
