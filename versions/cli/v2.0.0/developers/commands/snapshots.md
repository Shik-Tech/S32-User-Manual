# Snapshots

```text
s32 --loopback repl
s32> snapshot store knobs next Verse
s32> snapshot store knobs next Chorus
s32> snapshot list knobs
s32> snapshot recall knobs 1
s32> snapshot bank set knobs sync=1bar fade=1/4
```

`PART` is the bank: `knobs`, `lfos`, or `rack1`…`rack8` (RACK #1 … #8). `SLOT` is `1`–`32`, the pad number. See [Snapshots](/snapshots/) for how banks work on the S32.

## Slots

| Command | What it does |
|---------|--------------|
| `snapshot list [PART]` | Names, which slots are full, and the active one. Default `knobs` |
| `snapshot store PART [SLOT\|next] [NAME]` | Takes the current sound into a slot. A rack bank with no devices answers `BadRequest`; `Busy` during a fade |
| `snapshot recall PART SLOT` | Recalls a slot |
| `snapshot delete PART SLOT` | Asks first. Empties a slot. `Busy` during a fade |
| `snapshot rename PART SLOT NAME` | Renames without taking again |
| `snapshot copy\|swap\|move PART SRC DEST` | `Busy` during a fade. An empty source answers `NotFound` |

## Bank settings

`snapshot bank get PART` and `snapshot bank set PART sync=… fade=…` (each optional, at least one).

| | Values |
|---|---|
| `sync=` | `immediate`, `1bar`, `2bars`, `3bars`, `4bars` |
| `fade=` | `off`, `1/16`, `1/8`, `1/4`, `1/2`, `1bar`, `2bars`, `4bars` |

## IN SNAPSHOT

Each macro, each whole LFO and each whole device has an IN SNAPSHOT switch. So does each parameter. A rack or LFO recall, fade and morph moves a parameter only when both its switch and its owner's switch are **YES**. A knob recall writes through its macros: a macro that is on sets its parameter whatever the parameter's switches say. A take still stores everything. Recall doesn't change the switches.

| Command | Switch |
|---------|--------|
| `snapshot flags get device RACK DEV PARAM` / `set … in=yes\|no morph=yes\|no` | One device parameter: IN SNAPSHOT and ALLOW MORPH, each optional; the other bit keeps its value |
| `snapshot flags get lfo s32 INDEX PARAM` / `set … in=yes\|no morph=yes\|no` | The same for one LFO parameter |
| `snapshot parent get device RACK DEV` / `set … in=yes\|no` | A whole device |
| `snapshot parent get lfo s32 INDEX` / `set … in=yes\|no` | A whole LFO |
| `macro set POT MACRO in=yes\|no` | One macro |

## Morph

`snapshot morph get PART`, `snapshot morph set PART amount=N` and `snapshot morph clear PART` fade live toward the active slot. `amount=` runs `0`–`16383`. Above `0` it starts from what is playing now. `0` ends a running morph. `clear` drops it. While a recall fade is running the answer is `Busy`. See [Morph](/snapshots/morph).

In a fresh `s32 --loopback repl`:

```text
s32> snapshot store knobs next Verse
s32> snapshot morph set knobs amount=8192
morph set part=knobs amount=8192
s32> snapshot morph get knobs
part=knobs pinA=none pinB=1 morphing=1 amount=8192
s32> snapshot morph set knobs amount=0
morph set part=knobs amount=0
s32> snapshot morph get knobs
part=knobs pinA=none pinB=none morphing=0 amount=0
```

## Arrange

[Arrange](/snapshots/lanes) plays slots as a song: up to 256 rows, one column per bank.

`snapshot arrange get` prints `repeat`, `rate`, `length`, the playing row (`song-row`, which counts from `1`, like the row numbers on the screen), the armed **QUEUE** (`drop-row`, `none` when off), then each column.

`snapshot arrange set` changes only the keys you name:

| Key | Values |
|-----|--------|
| `repeat` | `loop` or `once` |
| `rate` | Bars per row, `1`–`128` |
| `length` | Rows, `1`–`256` |
| `knobs`, `lfos`, `rack1`…`rack8` | Slots `1`–`32` and `rest`, comma-separated. A short list fills the rest of the column with `rest` |

```text
s32> snapshot arrange set repeat=once rate=4 knobs=1,2,1,2 lfos=1,rest,4
```

A new project starts with **ARRANGE** off. `snapshot arrange set` writes the song, not that switch: turn **ARRANGE** on in the pencil tab on the panel, or the playhead stays parked and `jump` / `queue` do nothing.

`snapshot sequence get PART` and `snapshot sequence set PART rate=… [steps=…]` edit one column the same way.

| Command | What it does |
|---------|--------------|
| `snapshot jump ROW` | Moves the playhead to `ROW` (`1`–`256`) |
| `snapshot queue ROW` | Arms a **QUEUE** on `ROW`. `snapshot queue off` disarms it |

## Mapping

`macro set` and `extmap add` take `target=snapshot-slot` or `target=morph-amount` with `part=PART`. LFOs can't target either.

- **Snapshot slot:** the note or Program Change you learn is pad `01` and the next 31 numbers fill the pads after it. Add `pad=1`–`pad=32` to map that one pad only, for example `extmap add ch=3 cc=34 target=snapshot-slot part=knobs pad=5`; it then ignores the bank's number and the CC sweep. `min=` and `max=` are slots `1`–`32`, as on the screen. The knob skips empty slots. A number outside 1–32 on assignment gives the full 1–32.
- **Morph amount:** always `0`–`16383`. `min=` and `max=` are ignored. `invert=` still works. It always picks up.

`macro get` prints `target=` and `part=` after `lfo=` (`rack-bypass` has no `part=`); `lfoParam=` follows `lfo=` on every other row.

Both turn IN SNAPSHOT to **NO** for that macro. To make it a normal macro again, set a new `mode=` (or `rack=`, `device=`, `param=`, `lfoParam=`) without `target=`. IN SNAPSHOT comes back to **YES** and the range becomes the new mode's full range:

```text
s32> macro set 1 1 target=morph-amount part=knobs
s32> macro set 1 1 mode=cc msb=7
s32> macro get 1 1
```

## Output after a set

`snapshot sequence set`, `arrange set`, `flags set`, `parent set`, `bank set` and `morph set` print what they set, then the matching `get` ([A set prints the state it left](./session.md#a-set-prints-the-state-it-left)). `flags set`, `parent set` and `bank set` already printed that state line, so the same line shows twice:

```text
$ s32 --loopback snapshot parent set rack 1 in=no
parent rack=1 in=no
parent rack=1 in=no
$ s32 --loopback snapshot bank set knobs sync=1bar
bank part=knobs sync=1bar fade=off
bank part=knobs sync=1bar fade=off
```

With `--json`, lanes and Arrange columns are arrays with one number per step and `null` for a rest (`"steps":[1,2,null]`); an Arrange `drop-row` of `none` is `null`, `active` and the morph pins are `null` when there are none, and `snapshot list --json` has `"name":null` for an empty slot.
