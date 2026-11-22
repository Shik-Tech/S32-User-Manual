# Examples

Each block runs as written in one `s32 --loopback repl`. On an S32, start `s32 --usb repl` instead.

Prompts to give an AI agent, with checked commands: [Prompts and examples](../ai/prompts.md).

## Back up and restore a project

```text
s32> list
s32> pull internal 1 /tmp/p1.s32p --force
s32> push /tmp/p1.s32p
s32> push /tmp/p1.s32p --activate
s32> active
```

`pull` does not replace a file unless you add `--force`.

See [Push](./commands/projects.md#push).

## Map a knob to filter cutoff

Knob 05, macro 2, CC 74 on channel 3:

```text
s32> macro set 5 2 mode=cc msb=74 channel=3
s32> potname set 5 Cutoff
s32> macro get 5 2
```

See [Macros](./commands/macros.md) for every mode.

## Add an Arpeggiator

```text
s32> device add 1 arp
s32> device list 1
s32> device param set 1 2 rate=6
s32> device param get 1 2 rate
```

## Sweep the Arpeggiator gate with an LFO

Continue from the block above. `rack=` and `device=` are [ids, not positions](./commands/racks.md): use the rack id and the Arpeggiator's `id=` from the `device list 1` above (on the simulator, `100` and `1`):

```text
s32> lfo cfg set s32 1 wave=triangle freq=0.5 enabled=on
s32> lfo assign add 1 target=device rack=100 device=1 param=gate min=20 max=80
s32> lfo assign add 1 target=lfo targetLfo=2 param=depth min=0 max=1
s32> macro set 3 2 mode=cc msb=1
s32> lfo assign add 1 target=macro pot=3 macro=2
s32> lfo assign list
```

LFO 1 now also moves LFO 2's depth and knob 03, macro 2.

## Grab the screen

```text
s32> view nav s32-lfo index=3
s32> screen png /tmp/oled.png
```

## Watch the knobs

```bash
s32 --usb watch 8000
```

Prints knob events for eight seconds.

## Build a song in Arrange

```text
s32> snapshot store knobs next Verse
s32> snapshot store knobs next Chorus
s32> snapshot arrange set repeat=loop rate=4 length=16 knobs=1,2,1,2
s32> snapshot arrange get
```

New projects start with **ARRANGE** off, and USB writes the song but not that switch. Turn **ARRANGE** on in the pencil tab on the panel to start the playhead.

See [Arrange](./commands/snapshots.md#arrange).
