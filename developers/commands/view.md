# View, screen, events

```bash
s32 --loopback view state
s32 --loopback view nav racks
s32 --loopback view index 3
s32 --loopback view back
s32 --loopback screen png /tmp/oled.png
s32 --loopback watch 2000
```

## View

| Command | What it does |
|---------|--------------|
| `view state` | The screen, highlighted row, Back depth and context (rack, device, knob, macro, bank) |
| `view nav MENU [RACK [DEVICE]] [index=N\|row=N\|step=N\|layer=N]` | Opens a screen. `RACK` (`1`–`8`) and `DEVICE` (`1`–`10`) pick the rack and device it shows, or `none`. `index=N` highlights row `N` (from `1`); on a Step Seq or Euclid, `row=N` opens its **SETTINGS** row, `step=N` a Step Seq step and `layer=N` a Euclid layer (one target key per line) |
| `view index N` | Moves the highlight to row `N` (from `1`) |
| `view back` | Like pressing **Back** once |

`view nav` replaces the current screen without adding a Back step. Back from there goes to Perform. A row number past the end is an error and the screen stays.

```bash
s32 --loopback view nav perform
s32 --loopback view nav s32-lfo index=3
s32 --loopback view nav device-params 1 2
s32 --loopback view nav device-params 1 2 row=3
s32 --loopback view nav device-params 1 3 layer=2
```

`view nav device-params 1 2` opens rack 1, device 2. `index=` is the row, never the LFO, rack or device.

`row=`, `step=` and `layer=` land where the editor's "show on the S32" lands: `row=3` opens the Step Seq **RATE** row (BYPASS is `row=1`), `step=4` focuses step 4 on the Step Seq grid, `layer=2` focuses Euclid layer 2. They apply also when that screen is already open, and need firmware 1.1.4 or later (`s32 hello` prints `navTarget yes`); an older S32 answers `this S32 cannot open a setting row (firmware 1.1.4 or later)`. Use one of `index=`, `row=`, `step=`, `layer=` on a line.

Screens `view nav` opens:

`perform`, `main`, `settings`, `project-settings`, `clock`, `midi-settings`, `global-channel`, `middle-c`, `encoder-button`, `racks`, `devices`, `device`, `device-params`, `lfos`, `s32-lfo`, `projects`, `project-list`, `project-racks`, `snapshots`, `snapshot-parts`, `snapshot-racks`, `usb`

`snapshots` and `snapshot-parts` both open the SNAPSHOTS hub (KNOBS / RACKS / LFOS). The 4×8 bank screen (`snapshot-bank`) is not a public view.

Other screens (Arrange, editors, text entry) stay on the panel. Rename, add devices or load projects with their own commands (`rack rename`, `device add`, `project load`, …).

## Screen

`screen png FILE.png` saves what the screen shows as a PNG.

## Events

`watch [MS]` prints events for `MS` milliseconds (default 5000), including any queued by earlier commands in the same run. On `--usb`, turn a knob while it runs to see it, or change a Step Seq step on the panel: `EVT_STEPSEQ_PATTERN_CHANGED rack=1 device=2` says the editor should re-read that pattern.

## JSON

`view state --json` prints the open menu as one object (`menu`, `index`, `backStack`, `ctx`, `dialog`, `dialogId`, `dialogName`); `dialogName` is `null` when the dialog has no named text. `index` and the `ctx` numbers count from 1, as on the panel; a context the screen does not use is `null` (`none` in text), and `ctx.part` is the bank word (`knobs`, `lfos`, `rack1`…`rack8`). `view nav`, `view back`, `view index` and `dialog dismiss` take [`--dry-run`](./session.md#preview-before-a-change) and send nothing.
