# Simulator and scripts

## `--loopback`

`--loopback` runs the same protocol against a simulated S32 inside the s32 CLI process. No hardware needed. Use it to try commands, write examples and run CI. The s32 CLI never picks the simulator on its own: without `--loopback` it talks to your S32 on USB.

Every `--loopback` run prints `note: using the s32 simulator (--loopback), not your S32` on stderr; `-q` and `--json` leave it out.

- **Every run starts fresh** with three projects (Project 1–3) and one card project. A `macro set` in one run is gone in the next. Chain steps in a [REPL](./getting-started.md#interactive-session) or a script.
- **Real devices.** Every device type with its real parameters, defaults and ranges, as `schema --emit-schema-json` lists them. Values are rounded and clamped to each parameter's range, as on the S32. A range that follows a device's SYNC setting (Arpeggiator RATE, Delay DELAY TIME) is the full range here.

## `--script`

One command per line. `#` starts a comment. A script does not prompt, and `--script` does not imply `--yes`: a command that [asks first](./connecting.md#usb) needs `--yes` on the `s32` command line (`s32 --yes --script FILE`) or on its own line, otherwise that line fails with exit code 5 and the run stops. The run stops at the first line that fails and exits with that line's [exit code](./commands/session.md#exit-codes). `pause [MS]` waits (default 1000 ms).

```bash
s32 --loopback --script session.script
s32 --usb --script session.script
```

`session.script`:

```text
hello
list
macro get 1 1
macro set 5 2 mode=cc msb=74 channel=3
potname set 5 Cutoff
```

Words in a script line split on spaces, with no quoting. Use names without spaces.

## JSON in scripts and the REPL

Put `--json` before `--script` or `repl` and every line that succeeds prints exactly one JSON object (NDJSON), with no echo and no banner:

```bash
printf 'macro get 5 2\nrack list\nmacro set 5 2 channel=3\n' > j.script
s32 --loopback --json --script j.script
```

```text
{"pot":5,"macro":2,"mode":"disabled",...}
{"count":1,"max":8,"racks":[{"index":1,"id":100,"bypass":"off","in":"yes","devices":1,"name":"Rack 1"}]}
{"pot":5,"macro":2,"mode":"disabled","msb":0,"lsb":0,"channel":"3",...}
```

The third line is the read-back of `macro set`. A failing line prints one [JSON error](./commands/session.md#errors) on stderr with `line` and `text`, and the script stops with that line's exit code; in a REPL the session goes on. The rest is in [JSON output](./commands/session.md#json-output).

## Test fixtures

`S32_LOOPBACK_DROP_AFTER_WRITE=N` makes the simulator answer nothing after the Nth write has been answered Ok, so the next request times out (exit code 3). It tests what a program does when a change landed but could not be confirmed: `S32_LOOPBACK_DROP_AFTER_WRITE=1 s32 --loopback macro set 5 2 channel=3 --json` prints one error with `"class":"link"` and `"applied":true`.
