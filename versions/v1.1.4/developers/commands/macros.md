# Macros and knobs

Each of the 32 knobs has four [macros](/macros/). A macro turns knob motion into MIDI, a rack device parameter or an S32 LFO parameter.

```bash
s32 --loopback macro get 5 2
s32 --loopback macro set 5 2 mode=cc msb=74 channel=3
s32 --loopback potname get 5
s32 --loopback potname set 5 Cutoff
s32 --loopback knobmap get 1
s32 --loopback knobmap copy 4 10
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

`macro get KNOB MACRO`: knob `1`–`32`, macro `1`–`4`.

`macro set KNOB MACRO key=value …` changes only the keys you name:

| Key | Values |
|-----|--------|
| `mode` | A name from the table above |
| `msb` | CC number, program number, NRPN/RPN MSB, note, … |
| `lsb` | LSB, where the mode has one |
| `channel` | `global` or `1`–`16` |
| `min` / `max` | Range. 7-bit modes: `0`–`127`. `cc14`, `nrpn`, `rpn`, `pitch-bend`, `sysex`: `0`–`16383`. `mode=rack` and `target=rack-bypass`: `0`–`1`. `target=snapshot-slot`: `1`–`32`. `target=morph-amount`: `0`–`16383`. `mode=s32-lfo`: the setting's range (`enabled`, `sync`, `rate`, `depth` `0`–`1`; `wave` `1`–`17`; `phase` `0`–`360`; `offset` `-1`–`1`). `mode=device`: the parameter's range, except a time that changes with SYNC (see below) |
| `invert` | `on` / `off` |
| `outputs` | `none`, `usb`, `out1`, `out2`, or a `+` mix such as `usb+out1` |
| `pickup` | `on` / `off` |
| `in` | IN SNAPSHOT, `yes` / `no` |
| `rack` / `device` / `param` | `mode=device`: rack and device ids from `rack list` / `device list`, and a parameter name (`bypass`, `rate`, `gate`, …) or number from `schema`. `mode=rack`: the rack id from `rack list`; without `rack=` no rack is set. `param` can only be `bypass` |
| `sub` | `mode=device` on a Step Seq step: the step, `1`–`16`, or `none` |
| `lfo` / `lfoParam` | `mode=s32-lfo`: the LFO (`1`–`16`, or `none`) and its setting (`enabled`, `wave`, `rate`, `sync`, `phase`, `depth`, `offset`) |
| `name` | The label for a named control |

A rack, device or parameter that is not in the project exits `4` (`NotFound`) and changes nothing. Use the ids from `rack list` / `device list`, not positions, and a parameter that device has in `schema`.

On a time that changes with SYNC (Delay `delay-time`, Arpeggiator `rate`, Envelope `attack` / `hold` / `decay` / `release`), `min` and `max` are positions from `0` to `1` of the range it has in its current mode: `min=0 max=1` is the whole range on SYNCED and on FREE, and `min=0.5 max=1` stays the upper half when SYNC changes. `macro get` prints the positions. A value above `1` is read in the current unit (ticks or ms) and stored as its position.

```text
macro set 5 2 mode=cc msb=74 channel=3 outputs=usb+out1
macro set 1 3 mode=cc14 msb=1 lsb=33 channel=3
macro set 1 1 mode=device rack=100 device=1000 param=bypass
macro set 3 1 mode=s32-lfo lfo=1 lfoParam=depth
macro set 4 1 mode=disabled
```

Changing the mode, or picking a new target, without `min=`/`max=` gives the macro the full range of the new mode, as the panel does. A custom window such as CC `10`–`20` is dropped on `mode=pc` (`0`–`127`), `mode=cc14` (`0`–`16383`) or `mode=rack` (`0`–`1`). These keys also reset the range: `param=` in `mode=device` (the parameter's range), `lfoParam=` in an LFO mode (that setting's range), `rack=` in `mode=rack` (`0`–`1`), and `part=` with `target=snapshot-slot` / `target=morph-amount`. Naming the value the macro already has changes nothing.

These keys keep the window: `sub`, `rack` / `device` in `mode=device`, `channel`, `msb`, `lsb`, `invert`, `outputs`, `pickup`, `in`, `name`. In an LFO mode, an `lfo=` that changes the LFO index sets the LFO setting to `enabled` and its range `0`–`1`; with `lfoParam=`, the setting you name decides. Naming the LFO index the macro already has changes nothing, and `lfo=none` (no LFO) keeps the window and the setting.

Entering `mode=device` without `param=`:

- A macro that already holds a complete device target (a rack, a device and a parameter, `bypass` included) keeps it, and the S32 gives it that parameter's range.
- Any other macro (a fresh one, a Rack control, a snapshot or morph macro) leaves the target unset, with the range `0`–`127`. Name `rack=`, `device=` and `param=` to target a parameter.

Entering `mode=s32-lfo` without `lfoParam=` keeps the macro's LFO setting (`frequency` becomes `rate`, `amplitude` becomes `depth`, any setting an S32 LFO cannot target becomes `enabled`) with that setting's range.

Entering `mode=rack` drops a device target. With `rack=`, the knob controls that rack's BYPASS. Without it, no rack is set until you name one, as on the panel. The range is `0`–`1`. `device`, `sub`, `cv`, `lfo` and `lfoParam` are refused with `mode=rack`.

Name both `min=` and `max=` to set your own window on the same line. Naming one takes the other end from the new range (`mode=pc min=5` gives `5`–`127`). The exception is a device parameter, whose range only the S32 knows: there, naming one of the two exits `2` (`name both min= and max=, or neither, when the device parameter changes`) and changes nothing. `--dry-run` shows the range the line would set, `parameter-range` for a device parameter.

`target=snapshot-slot` and `target=morph-amount` with `part=` turn a macro into a snapshot picker or a morph knob. See [Snapshots](./snapshots.md#mapping).

`target=rack-bypass rack=ID` maps a macro to a rack's BYPASS, the same target
the panel's rack Options use:

```text
macro set 4 1 target=rack-bypass rack=100
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
$ s32 --loopback potname set 5 Cutoff
pot 5 renamed
pot 5 name=Cutoff
$ s32 --loopback macro set 5 2 channel=3
macro 5/2 updated
pot 5 macro 2: mode=disabled msb=0 lsb=0 channel=3 min=0.000 max=127.000 invert=off outputs=none pickup=off in=yes rack=0 device=0 param=0 sub=none cvOut=none lfo=none lfoParam=none name=
```

With `--json`, `macro get` and `macro set` print the same object, keyed by the names `macro set` takes (`mode`, `msb`, `channel`, `outputs`, `in`, `lfo`, `lfoParam`, `cvOut`, ...). `target` and `part` are fields: `null` unless the macro is a snapshot-slot, morph-amount or rack-bypass target. `lfoParam` is the LFO destination field. `knobmap get --json` lists the knobs with their first macro.

```bash
s32 --loopback macro get 5 2 --json
s32 --loopback macro set 5 2 channel=3 msb=74 --dry-run
```
