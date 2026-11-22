# Racks and devices

```text
s32 --loopback repl
s32> rack list
s32> rack rename 1 Lead
s32> device add 1 step-seq
s32> device list 1
s32> stepseq set 1 2 step 1 active=1 note=60 vel=100
s32> stepseq get 1 2
s32> rack duplicate 1 2
s32> rack set 2 bypass=on
```

`RACK` and `DEV` are positions: rack `1` is the first [rack](/racks/), device `1` the first device in it, as on the panel. Ids (`rack list`, `device list`) are used only by `lfo assign` and `macro set … mode=device`.

## Racks

| Command | What it does |
|---------|--------------|
| `rack list` | Position, id, bypass, device count and name of each rack. An empty slot's id prints `-` |
| `rack rename RACK NAME` | Keeps the first 10 characters |
| `rack set RACK bypass=on\|off` | Bypasses the whole rack |
| `rack clear RACK` | Asks first. Removes every device, turns bypass off and sets `in=yes` |
| `rack duplicate SRC DST` | Asks first. Overwrites the destination rack; the copy keeps the source's `bypass` and `in` |

## Devices

| Command | What it does |
|---------|--------------|
| `device list RACK` | Position, type, id and name of each device. A rack whose first device is not MIDI IN or CLOCK adds `needs MIDI IN or CLOCK first`; `--json` reports `"needsStart": true` |
| `device add RACK TYPE` | Adds a [device](/devices/) to the rack. `TYPE` is a name (`midi-in`, `midi-out`, `clock`, `arp`, `step-seq`, …) or its number from `schema`. MIDI IN goes first; a CLOCK the rack needs also goes first, otherwise before MIDI OUT |
| `device remove RACK DEV` | Removes the device, including the first MIDI IN or CLOCK. Its LFO assignments and external MIDI mappings are removed; a knob macro on it keeps its rack and shows SET TARGET. |
| `device move RACK FROM TO` | Reorders inside the rack. MIDI IN stays first, MIDI OUT last, and a CLOCK on top of a rack with no MIDI IN stays on top |
| `device param get RACK DEV PARAM` | Value and range |
| `device param set RACK DEV PARAM=VALUE [PARAM=VALUE …]` | One or more settings in one sentence. Rounded to a whole number and clamped to the range |
| `device param action RACK DEV PARAM` | Presses that row's button, like Select on the panel: `randomize` (Step Seq), `record` or `reset` (Looper), or a `PARAM` number. A normal row answers `BadRequest` |
| `device host-device RACK DEV` | Which USB Host device a MIDI IN or MIDI OUT uses: `ALL` or a device name |
| `device host-device RACK DEV host=all\|N\|NAME` | Picks ALL, or a device from `host-device list` by number or name. The name runs to the end of the line, so it can hold spaces |

An empty rack takes **MIDI IN** or **CLOCK** first. A device that can't go there, or a full rack, answers `BadRequest`. `NoSpace` means the S32 has no room for another device. Loading a project keeps every device: a rack with its MIDI IN or CLOCK lower down opens with that start device first, the other devices in the same order, and MIDI OUT last.

`PARAM` is a parameter name (`bypass`, `mode`, `rate`, `gate`, `clock-division`, `port`, `host-device`, …) or its number from `schema`, not the row on the screen.

## USB Host devices

`host-device list` prints the devices this project knows on the [USB HOST port](/concepts/routing#usb-host), numbered as **HOST DEVICE** counts them (`0` is **ALL**). A device you unplugged stays listed, marked `(unplugged)`. `stored` is the name exactly as the project keeps it, in the case the device reported; it is left out when the device has no name. `device host-device` matches the panel text, not `stored`, so pick a second identical device by its number.

```text
s32> host-device list
host devices: 4 of 8
  0 ALL
  1 KEYBOARD  (1209:0001 at 011)  stored "Keyboard"
  2 KEYBOARD (2) (unplugged)  (1209:0001 at 012)  stored "Keyboard"
  3 SYNTH (unplugged)  (1209:0002 at 021)  stored "Synth"
  4 1209:0003  (1209:0003 at 031)
s32> device host-device 1 1 host=keyboard
rack 1 device 1 HOST DEVICE = KEYBOARD
```

**HOST DEVICE** only does something while that MIDI IN or MIDI OUT has **PORT** set to **USB HOST**.

IN SNAPSHOT and ALLOW MORPH for device parameters are under [`snapshot flags`](./snapshots.md#in-snapshot).

## Step Seq

`stepseq get RACK DEV` prints all 16 steps. `DEV` must be a Step Seq.

`stepseq set RACK DEV step N key=value …` changes one step (`1`–`16`). Keys: `active`, `note`, `vel`, `gate`, `ch`, `prob`.

## Output after a set

`rack rename` and `rack set` print the rack's row, `device param set` one line per parameter, `device host-device ... host=` the HOST DEVICE and `stepseq set` the pattern ([A set prints the state it left](./session.md#a-set-prints-the-state-it-left)):

```text
$ s32 --loopback rack rename 1 Lead
rack 1 renamed
  [1] id=100  bypass=off in=yes devices=1 Lead
$ s32 --loopback device param set 1 1 port=4
rack 1 device 1 param port (149) set
rack 1 device 1 param port (149) = 4.000 (0.000..127.000)
$ s32 --loopback device host-device 1 1 host=all
rack 1 device 1 HOST DEVICE = ALL
rack 1 device 1 HOST DEVICE = ALL
```

`device host-device` already read the HOST DEVICE back, so the same line prints twice: your confirmation, then the read-back.

With `--json`, `device param get` and `device param set` print one object with a `params` array (one entry per parameter, in the order given). `device host-device --json` is `{"rack":1,"device":1,"host":"ALL","value":0,"connected":true,"listed":true}`: `host` is the stored name without ` (unplugged)` (or `null` for a number the list does not hold), `connected` is `false` exactly when the text says `(unplugged)`, and `listed` is `false` exactly for `(not listed)`.
