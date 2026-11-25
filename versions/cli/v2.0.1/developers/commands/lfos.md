# LFOs

```text
s32 --loopback repl
s32> lfo cfg get s32 3
s32> lfo cfg set s32 3 wave=sine freq=3.75 enabled=on
s32> macro set 5 2 mode=cc msb=1
s32> lfo assign add 3 target=macro pot=5 macro=2 min=-0.25 max=0.25
s32> lfo assign list
s32> lfo assign remove 1
```

A macro whose mode takes no LFO (disabled, device, rack control, S32 LFO) is refused with `BadRequest`; set a MIDI-message mode first.

`DOMAIN` is `s32` (the S32's own [LFOs](/lfos/)). `INDEX` is the LFO, `1`–`16`.

## Settings

`lfo cfg get DOMAIN INDEX` prints the LFO's settings (`retrigger=` included), then its IN SNAPSHOT (`liveOwned=`) and ALLOW MORPH (`allowMorph=`) bits as hex masks. A stored reserved retrigger value prints `reserved(N)`. `lfo cfg set` never changes those bits. Use [`snapshot flags` and `snapshot parent`](./snapshots.md#in-snapshot).

`lfo cfg set DOMAIN INDEX key=value …` changes only the keys you name; there is no positional frequency form. `freq=HZ` sets the frequency and leaves `enabled` alone. Values outside the LFO's range are refused.

| Key | S32 |
|-----|-----|
| `enabled` | `on` / `off` |
| `wave` | see below |
| `freq` | Hz |
| `bpm` | Clock division, used with `sync=on` |
| `sync` | `on` / `off` |
| `retrigger` | `off` / `enable` / `start` / `both` (default `start`) |
| `phase` | Degrees |
| `offset` | yes |
| `amp` | `0`–`1` |
| `min` / `max` | yes |

Waves: `sine`, `triangle`, `square`, `saw`, `fold`, `s-h`, `exponential`, `stepped`, `double-sine`, `rev-saw`, `bell`, `double-triangle`, `smooth-random`, `trapezoid`, `pump`, `rounded-square`, `half-sine`.

## Assignments

`lfo assign add LFO target=… …` adds a row. `lfo assign set ROW LFO target=… …` rewrites one. `lfo assign remove ROW` deletes one. `lfo assign list` shows the row numbers.

| `target=` | Moves | Address | `param=` |
|-----------|-------|---------|----------|
| `device` | A rack device parameter | `rack=ID device=ID` (ids from `rack list` / `device list`) | Parameter name (`bypass`, `rate`, `gate`, …) or number |
| `lfo` | Another LFO in the same domain | `targetLfo=LFO` (`1`–`16`) | LFO parameter (`enabled`, `wave`, `rate`, `sync`, `depth`, `offset`, `phase`) |
| `macro` | A knob macro | `pot=KNOB macro=MACRO` | — |

An LFO can't modulate itself or close a loop of LFOs (1 → 2 → 1). Those rows are refused.

A rack, device or parameter that is not in the project exits `4` (`NotFound`) and changes nothing. Use the ids from `rack list` / `device list`, not positions, and a parameter that device has in `schema`.

### Range

For `device` and `lfo` targets, `min` and `max` are values in the target's own range. Values outside it are clamped to the edge. On S32 targets `min` above `max` runs the sweep the other way.

A time that changes with SYNC (Delay `delay-time`, Arpeggiator `rate`, Envelope `attack` / `hold` / `decay` / `release`) is the exception: `min` and `max` are positions from `0` to `1` of the range it has in its current mode, so the row keeps working when SYNC changes. `lfo assign list` prints them that way (`range=0.250..0.750`). A value above `1` is read in the current unit (ticks or ms) and stored as its position: `min=6 max=96` on a SYNCED Arpeggiator is `1/16` … `1/1`.

For `macro` targets, `min` and `max` are the BELOW and ABOVE window around the knob, as fractions of the macro's range from `-1` to `1`. The default is `-0.5` / `0.5`. `min=0 max=0.4` only rises above the knob. Capability bit `1024` in `hello` means the firmware uses this window.

## Output after a set

`lfo cfg set` prints the LFO's settings after its confirmation, and `lfo assign add` and `lfo assign set` print the row they wrote, with no list header ([A set prints the state it left](./session.md#a-set-prints-the-state-it-left)):

```text
$ s32 --loopback lfo cfg set s32 1 freq=2 wave=triangle
lfo s32 1 updated
s32 lfo 1: enabled=on wave=triangle freq=2.000 bpm=1.000 amp=1.000 offset=0.000 sync=on retrigger=start liveOwned=0x0000 allowMorph=0x9000
$ s32 --loopback lfo assign add 1 target=macro pot=2 macro=1
added assignment 1 (1 total)
  [1] lfo=1 target=macro pot=2 macro=1 range=-0.500..0.500
```

With `--json`, `lfo cfg get` also reports `phase` in degrees (`0`–`360`, as `phase=` takes it); the text output does not print it. `amp` is `null` for a CV LFO and `depth` is `null` for an s32 LFO. In `lfo assign list --json` each row has `rack`, `device`, `param`, `pot`, `macro` and `targetLfo`, `null` where the target does not use them, and `range=a..b` is `min` and `max`.
