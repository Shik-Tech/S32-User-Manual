# Prompts and examples

Prompts you can give an AI agent, each with the commands it runs on the s32 CLI and the lines it should see. Every command here is checked on the simulator. For people who type the commands themselves, see [Examples](../developers/examples.md). To set the agent up first, start with [Set up your agent](./setup.md).

Commands count from 0: knob 01 is `0`, macro 1 is `0`, rack 1 is `0`, and a device's position in `device list` is its number (MIDI IN is `0`). Rack and device **ids** (`id=` in `rack list` and `device list`) are not positions.

- A name in angle brackets, like `<RACK_ID>` or `<EUCLID_ID>`, is an id: the agent fills it in from the `device list` it ran just before. Do not type it as written.
- Every example starts from a fresh simulator. Your S32's racks, ids and names differ: the agent reads them first (`s32 device list 0`, `s32 rack list`) and uses yours.
- The lines under each block are the ones to check. The commands print more, and the read-back after each change shows the new state.
- On the S32, `device add` puts a new device before MIDI OUT. If the rack already has a Clock or a MIDI OUT, the agent skips adding it.
- A new MIDI IN and a new MIDI OUT use **PORT USB**. To use a MIDI jack, ask for it: `port=1` is MIDI1.
- To hear anything in time, start the S32's clock (press the encoder) or start it from your DAW.
- **Parameter numbers and values** used on this page. The agent finds them on your S32 with `s32 schema --emit-schema-json`, which lists each parameter's `id`, `name` and range. Plain names work for `rate`, `gate` and `port`.

| Device | Parameter | Number | Values used here |
|---|---|---|---|
| Arpeggiator | RATE | `rate` | `6` = 1/16 on SYNCED (steps are 24 per quarter note: 96 = 1/1, 12 = 1/8, 6 = 1/16) |
| Arpeggiator | GATE | `gate` | % of the step, 10–200 |
| MIDI IN, MIDI OUT | PORT | `port` | `0` USB, `1` MIDI1, `2` MIDI2, `3` USB HOST |
| Harmony | 1ST, 2ND | `87`, `88` | semitones, `4` = +4, `7` = +7 |
| Scaler | KEY, QUANTIZE, SCALE | `18`, `19`, `20` | KEY `0` C … `9` A; QUANTIZE `0` DOWN; SCALE = its number in the Scaler's list (`2` MAJOR, `5` PENTATONIC MINOR) |
| Seed | RATE, ROOT NOTE | `rate`, `154` | RATE `6` = 1/8; ROOT NOTE is a MIDI note, `57` = the A below middle C |
| Random | PARAMETER, POSITIVE, NEGATIVE | `25`, `27`, `28` | PARAMETER `0` VELOCITY; amounts 0–127 |
| Envelope | TRIGGER, DEST, ATTACK, DECAY, SUSTAIN, RELEASE, DEST MSB | `171`, `168`, `161`, `163`, `164`, `165`, `169` | TRIGGER `2` POLY; DEST `6` POLY AT (CC is `0`, the default); times on SYNCED in steps of 24 per quarter (`48` = 1/2, `24` = 1/4, `12` = 1/8, `6` = 1/16); SUSTAIN %; DEST MSB = the CC number |
| Delay | FEEDBACK, DRY/WET | `32`, `33` | % |
| Euclid | ENABLED, STEPS, PULSES, SHIFT, RATE, NOTE, GATE, CHANNEL, VELOCITY | layer 1 `50`–`58`, layer 2 `59`–`67`, layer 3 `68`–`76`, layer 4 `77`–`85`, in this order | ENABLED `1` = on; RATE `4` = 1/16 per step (`2`, the default, = 1/4); NOTE is a MIDI note, `45` = A2; GATE % of the step |

## Quick starters

Three prompts to try first. Each shows the commands an agent runs and the lines it should see. The simulator starts fresh on every run, so your S32's racks and ids differ: the agent reads yours first.

### Q1 · What's on my S32?

**Goal:** know which project is loaded and what rack 1 holds before you change anything.

**Prompt:**

<!-- s32-skip: prompt for the agent, not a command -->
```text
Is my S32 connected? Tell me which project is loaded and what's in rack 1.
```

<!-- s32-check -->
```bash
s32 hello
s32 active
s32 rack list
s32 device list 0
```

**Output**

<!-- s32-output -->
```text
volume=0 uid=1 dirty=0 dirtyGen=0 name=Project 1
1/8 racks
  [0] id=100  bypass=off in=yes devices=1 Rack 1
rack 0 (id 100): 1 device(s)
  [0] type=midi-in      id=1000  params=3   Device 1
```

**What you hear:** nothing changes. The agent reports the project, the racks and rack 1's devices.

### Q2 · Arpeggiate my keyboard

**Goal:** hold a chord and hear it played note by note in sixteenths (the [Quick Start](../getting-started/quick-start.md), by prompt).

**Prompt:**

<!-- s32-skip: prompt for the agent, not a command -->
```text
In rack 1, arpeggiate what I play in sixteenth notes. My keyboard is on the MIDI IN jack and my synth on MIDI OUT 1.
```

<!-- s32-check -->
```bash
s32 device list 0
s32 device add 0 clock
s32 device add 0 arp
s32 device add 0 midi-out
s32 device param set 0 0 port=1
s32 device param set 0 2 rate=6
s32 device param set 0 3 port=1
s32 device list 0
```

**Output**

<!-- s32-output -->
```text
rack 0 device 0 param port (149) set
rack 0 device 2 param rate (3) set
rack 0 device 3 param port (149) set
rack 0 (id 100): 4 device(s)
  [0] type=midi-in      id=1000  params=3   Device 1
  [1] type=clock        id=1     params=2   Device 2
  [2] type=arp          id=2     params=9   Device 3
  [3] type=midi-out     id=3     params=3   Device 4
```

**What you hear:** press the encoder to start the clock and hold a chord. Your synth plays it one note at a time, in sixteenth notes at the project tempo. Let go and it stops.

### Q3 · Filter cutoff on knob 01

**Goal:** sweep your synth's filter from knob 01, labelled CUTOFF.

**Prompt:**

<!-- s32-skip: prompt for the agent, not a command -->
```text
Make knob 01 control my synth's filter cutoff, CC 74 on channel 1 out of MIDI OUT 1, and name it CUTOFF.
```

<!-- s32-check -->
```bash
s32 knobmap get 0
s32 macro set 0 0 mode=cc msb=74 channel=1 min=0 max=127 outputs=out1
s32 potname set 0 CUTOFF
s32 knobmap get 0
```

**Output**

<!-- s32-output -->
```text
macro 0/0 updated
pot 0 renamed
  pot  0 CUTOFF           macro0 mode=cc msb=74
```

**What you hear:** turn knob 01 and the filter opens and closes. Most synths use CC 74 for cutoff; if yours does not, ask the agent for its number.

## Rack setups

### R1 · Arpeggio into echoes

**Goal:** a sixteenth-note arpeggio that echoes in eighth notes.

**Prompt:**

<!-- s32-skip: prompt for the agent, not a command -->
```text
In rack 1, arpeggiate what I play in sixteenths and echo every note.
```

<!-- s32-check -->
```bash
s32 device list 0
s32 device add 0 clock
s32 device add 0 arp
s32 device add 0 delay
s32 device add 0 midi-out
s32 device param set 0 2 rate=6
s32 device list 0
```

**Output**

<!-- s32-output -->
```text
rack 0 device 2 param rate (3) set
rack 0 (id 100): 5 device(s)
  [0] type=midi-in      id=1000  params=3   Device 1
  [1] type=clock        id=1     params=2   Device 2
  [2] type=arp          id=2     params=9   Device 3
  [3] type=delay        id=3     params=10  Device 4
  [4] type=midi-out     id=4     params=3   Device 5
```

**What you hear:** each arpeggio note comes back an eighth note later, five times, each echo half as loud as the one before. These are the Delay's own starting values: DELAY TIME 1/8, REPETITIONS 5, FEEDBACK 50%, DRY/WET 50%.

### R2 · Chords in key from one finger

**Goal:** one note in, a triad of C major out.

**Prompt:**

<!-- s32-skip: prompt for the agent, not a command -->
```text
In rack 1, turn every note I play into a chord that stays in C major.
```

<!-- s32-check -->
```bash
s32 device list 0
s32 device add 0 harmony
s32 device add 0 scaler
s32 device add 0 midi-out
s32 device param set 0 1 87=4 88=7
s32 device param set 0 2 18=0 19=0 20=2
s32 device list 0
```

**Output**

<!-- s32-output -->
```text
rack 0 device 1 param 87 (87) set
rack 0 device 1 param 88 (88) set
rack 0 device 2 param 18 (18) set
rack 0 device 2 param 19 (19) set
rack 0 device 2 param 20 (20) set
rack 0 (id 100): 4 device(s)
  [0] type=midi-in      id=1000  params=3   Device 1
  [1] type=harmony      id=1     params=10  Device 2
  [2] type=scaler       id=2     params=16  Device 3
  [3] type=midi-out     id=3     params=3   Device 4
```

**What you hear:** HARMONY adds a major third and a fifth (+4, +7). SCALER on C MAJOR with QUANTIZE DOWN pulls any note outside the key down a semitone. So every white key gives the chord of its scale degree: C gives C E G, D gives D F A, E gives E G B, F gives F A C, G gives G B D, A gives A C E, B gives B D F.

### R3 · A melody that writes itself, in key

**Goal:** a generated eighth-note melody in A minor pentatonic with human-sounding dynamics.

**Prompt:**

<!-- s32-skip: prompt for the agent, not a command -->
```text
In rack 1, generate an eighth-note melody in A minor pentatonic, starting on the A below middle C, and vary its velocity a little.
```

<!-- s32-check -->
```bash
s32 device list 0
s32 device add 0 clock
s32 device add 0 seed
s32 device add 0 scaler
s32 device add 0 random
s32 device add 0 midi-out
s32 device param set 0 2 rate=6 154=57
s32 device param set 0 3 18=9 19=0 20=5
s32 device param set 0 4 25=0 27=15 28=25
s32 device list 0
```

**Output**

<!-- s32-output -->
```text
rack 0 device 2 param rate (3) set
rack 0 device 2 param 154 (154) set
rack 0 device 3 param 18 (18) set
rack 0 device 3 param 19 (19) set
rack 0 device 3 param 20 (20) set
rack 0 device 4 param 25 (25) set
rack 0 device 4 param 27 (27) set
rack 0 device 4 param 28 (28) set
rack 0 (id 100): 6 device(s)
  [0] type=midi-in      id=1000  params=3   Device 1
  [1] type=clock        id=1     params=2   Device 2
  [2] type=seed         id=2     params=15  Device 3
  [3] type=scaler       id=3     params=16  Device 4
  [4] type=random       id=4     params=5   Device 5
  [5] type=midi-out     id=5     params=3   Device 6
```

**What you hear:** start the clock and a melody of eighth notes plays by itself.

- The notes span two octaves up from that A (NOTE RANGE 24, the default) and all fall on A, C, D, E and G.
- About one step in ten rests (DENSITY 90%), and the melody slowly mutates (CHANCE 50%).
- Velocities vary between 75 and 115: Seed's fixed 100, moved −25 to +15 by Random.
- To keep a melody you like, ask the agent to turn Seed's LOCK on.

## Modulation

### M1 · A breathing arpeggio, with a knob for how much

**Goal:** an LFO stretches the arpeggio's notes from clipped to smooth over two bars, and knob 02 sets how far.

**Prompt:**

<!-- s32-skip: prompt for the agent, not a command -->
```text
In rack 1, arpeggiate in sixteenths. Let LFO 1 slowly move the arpeggiator's note length between 25% and 90% over two bars, and give me knob 02, named BREATH, for how much it moves.
```

**Ids:** `<RACK_ID>` is the `id=` of rack 1 in `s32 rack list`, and `<ARP_ID>` is the Arpeggiator's `id=` in `s32 device list 0`. On the simulator they are `100` and `2`. Yours differ.

<!-- s32-check -->
```bash
s32 device list 0
s32 device add 0 clock
s32 device add 0 arp
s32 device add 0 midi-out
s32 device list 0
s32 device param set 0 2 rate=6
s32 lfo cfg get s32 0
s32 lfo cfg set s32 0 wave=sine sync=on bpm=0.5 enabled=on
s32 lfo assign add 0 target=device rack=<RACK_ID> device=<ARP_ID> param=gate min=25 max=90
s32 macro set 1 0 mode=s32-lfo lfo=0 lfo-param=depth min=0 max=1
s32 potname set 1 BREATH
s32 lfo cfg get s32 0
s32 lfo assign list
s32 knobmap get 1
```

**Output**

<!-- s32-output -->
```text
rack 0 (id 100): 4 device(s)
  [0] type=midi-in      id=1000  params=3   Device 1
  [1] type=clock        id=1     params=2   Device 2
  [2] type=arp          id=2     params=9   Device 3
  [3] type=midi-out     id=3     params=3   Device 4
rack 0 device 2 param rate (3) set
lfo s32 0 updated
added assignment 0 (1 total)
macro 1/0 updated
pot 1 renamed
s32 lfo 0: enabled=on wave=sine freq=1.000 bpm=0.500 amp=1.000 offset=0.000 sync=on retrigger=start liveOwned=0x0000 allowMorph=0x9000
1 assignment(s)
  [0] lfo=0 target=device rack=100 device=2 param=gate range=25.000..90.000
  pot  1 BREATH           macro0 mode=s32-lfo msb=1
```

**What you hear:** with the clock running, hold a chord. The arpeggio's notes swell from short and clipped to smooth and back, once every two bars (`bpm=0.5` is one cycle per 2 bars). Turn BREATH left and the note length stops moving; turn it right for the full 25–90% sweep.

### M2 · A filter pluck on every note

**Goal:** every note snaps your synth's filter open and lets it fall, like a plucked string.

**Prompt:**

<!-- s32-skip: prompt for the agent, not a command -->
```text
In rack 1, add an envelope that opens my synth's filter (CC 74) on every note: instant attack, an eighth-note decay to a low sustain, and a short release.
```

<!-- s32-check -->
```bash
s32 device list 0
s32 device add 0 clock
s32 device add 0 envelope
s32 device add 0 midi-out
s32 device param set 0 2 169=74 161=0 163=12 164=20 165=6
s32 device list 0
```

**Output**

<!-- s32-output -->
```text
rack 0 device 2 param 169 (169) set
rack 0 device 2 param 161 (161) set
rack 0 device 2 param 163 (163) set
rack 0 device 2 param 164 (164) set
rack 0 device 2 param 165 (165) set
rack 0 (id 100): 4 device(s)
  [0] type=midi-in      id=1000  params=3   Device 1
  [1] type=clock        id=1     params=2   Device 2
  [2] type=envelope     id=2     params=15  Device 3
  [3] type=midi-out     id=3     params=3   Device 4
```

**What you hear:** with the clock running, each note pushes the cutoff (CC 74) to the top at once. Over an eighth note it falls to a fifth of the way open while you hold the key. When you let go, it closes over a sixteenth. DEST CC, DEPTH 100%, OFFSET 50% and TRIGGER MONO are the Envelope's own starting values. It sits on SYNCED, so its times follow the tempo; that is why the rack has a Clock.

### M3 · Every note of a chord blooms on its own

**Goal:** each note you hold gets its own envelope, sent as poly aftertouch. Notes in a chord swell and fade independently.

**Prompt:**

<!-- s32-skip: prompt for the agent, not a command -->
```text
In rack 1, give every note I hold its own envelope, sent as poly aftertouch: swell in over a quarter note, settle over a half note to 60%, and fade over a quarter note when I let go.
```

<!-- s32-check -->
```bash
s32 device list 0
s32 device add 0 clock
s32 device add 0 envelope
s32 device add 0 midi-out
s32 device param set 0 2 171=2 168=6
s32 device param set 0 2 161=24 163=48 164=60 165=24
s32 device list 0
```

**Output**

<!-- s32-output -->
```text
rack 0 device 2 param 171 (171) set
rack 0 device 2 param 168 (168) set
rack 0 device 2 param 161 (161) set
rack 0 device 2 param 163 (163) set
rack 0 device 2 param 164 (164) set
rack 0 device 2 param 165 (165) set
rack 0 (id 100): 4 device(s)
  [0] type=midi-in      id=1000  params=3   Device 1
  [1] type=clock        id=1     params=2   Device 2
  [2] type=envelope     id=2     params=15  Device 3
  [3] type=midi-out     id=3     params=3   Device 4
```

**What you hear:** with the clock running, hold a chord, then add a note. Each note's aftertouch rises over a quarter note, falls over a half note to 60% while you hold it, and fades over a quarter note after you let go of that key. The others carry on. The new note starts its own swell while the held ones stay where they are.

This needs a synth that responds to poly aftertouch, for example with poly aftertouch mapped to its filter or volume. On a synth that ignores poly aftertouch you hear the notes unchanged.

## Mappings

### K1 · One knob builds a bass groove

**Goal:** a bass line in A minor that plays itself in sixteenths, and one knob that makes it busier.

**Prompt:**

<!-- s32-skip: prompt for the agent, not a command -->
```text
In rack 1, play a bass groove in A minor in sixteenths, and give me knob 01, named BUILD, that makes it busier.
```

**Ids:** `<RACK_ID>` is the `id=` of rack 1 in `s32 rack list`, and `<EUCLID_ID>` is the Euclid's `id=` in `s32 device list 0`. On the simulator they are `100` and `2`. Yours differ.

<!-- s32-check -->
```bash
s32 device list 0
s32 device add 0 clock
s32 device add 0 euclid
s32 device add 0 midi-out
s32 device list 0
s32 device param set 0 2 54=4 55=45 56=40 58=120
s32 device param set 0 2 59=1 61=3 62=2 63=4 64=57 65=50 67=95
s32 device param set 0 2 68=1 70=1 71=7 72=4 73=55 74=50 76=85
s32 device param set 0 2 77=1 79=1 80=11 81=4 82=60 83=40 85=80
s32 macro set 0 0 mode=device rack=<RACK_ID> device=<EUCLID_ID> param=61 min=3 max=9
s32 macro set 0 1 mode=device rack=<RACK_ID> device=<EUCLID_ID> param=70 min=1 max=5
s32 macro set 0 2 mode=device rack=<RACK_ID> device=<EUCLID_ID> param=56 min=40 max=90
s32 potname set 0 BUILD
s32 macro get 0 0
s32 macro get 0 1
s32 macro get 0 2
s32 knobmap get 0
```

**Output**

<!-- s32-output -->
```text
rack 0 (id 100): 4 device(s)
  [0] type=midi-in      id=1000  params=3   Device 1
  [1] type=clock        id=1     params=2   Device 2
  [2] type=euclid       id=2     params=37  Device 3
  [3] type=midi-out     id=3     params=3   Device 4
rack 0 device 2 param 56 (56) = 40.000 (0.000..200.000)
rack 0 device 2 param 61 (61) = 3.000 (1.000..16.000)
rack 0 device 2 param 70 (70) = 1.000 (1.000..16.000)
macro 0/0 updated
macro 0/1 updated
macro 0/2 updated
pot 0 renamed
pot 0 macro 0: mode=device rack=100 device=2 param=61
pot 0 macro 1: mode=device rack=100 device=2 param=70
pot 0 macro 2: mode=device rack=100 device=2 param=56
  pot  0 BUILD            macro0 mode=device msb=0
```

**What you hear:** start the clock and a bass line plays by itself, nothing played in. The Euclid's four layers each place a few notes on a 16-step bar, one step per sixteenth: layer 1 the root A2 on every beat, layer 2 the octave A3, layer 3 the minor seventh G3, layer 4 a single C4 near the end of the bar. With knob 01 left the groove is tight: the root on the beats, short, with a few octave pops. Turning it right adds octave and seventh hits and lengthens the root notes, until the line is busy and interlocking. `macro get` prints a parameter by its number: `61` is layer 2 PULSES, `70` is layer 3 PULSES and `56` is layer 1 GATE.


- [Overview](./index.md) — What an agent can do with your S32, in three steps
- [Set up your agent](./setup.md) — Paste one block; the agent installs, checks and saves the rules
- [Change safely](./safe-changes.md) — Read, preview, ask, change, check
- [For agents](./for-agents.md) — What agents read: llms.txt, Markdown copies, finding commands

