# Projects and storage

```bash
s32 --loopback list
s32 --loopback active
s32 --loopback storage
# replaces /tmp/p1.s32p if it is there
s32 --loopback pull internal 1 /tmp/p1.s32p --force
s32 --loopback push /tmp/p1.s32p
s32 --loopback push /tmp/p1.s32p --volume sd
```

`VOLUME` is `internal` (the S32's own storage) or `sd` (the microSD card). `UID` is the number `list` prints. See [Projects](/projects/) for how projects work on the S32.

## Library

| Command | What it does |
|---------|--------------|
| `list` | Every project on both volumes. `*` marks the one you are playing |
| `active` | The project you are playing, and whether it has unsaved changes |
| `storage` | Projects, capacity and free space per volume, in bytes, and the running job |
| `pull VOLUME UID FILE.s32p [--force]` | Copies a project to a file and checks it. A damaged copy is not written. An existing file is not replaced unless you add `--force` |
| `push FILE.s32p [--volume internal\|sd] [--activate]` | Stores the file as a new project (default `internal`). `--activate` also loads it |
| `project create VOLUME NAME [--seed empty\|daw]` | New project |
| `project delete VOLUME UID` | |
| `project rename VOLUME UID NAME` | |
| `project duplicate VOLUME UID NAME [--to internal\|sd]` | Copies a project. `--to` puts the copy on the other volume. `job HANDLE` then prints the copy's volume and uid |
| `project load VOLUME UID` | Replaces the project you are playing |
| `project save [--wait]` | Saves the project you are playing in place. `--wait` returns once it is stored and prints `elapsed_ms` |
| `project save-as VOLUME NAME [--activate] [--wait]` | Saves the project you are playing, unsaved changes included, as a new project |

`pull --dry-run` checks the project and says whether the file would be created or replaced. `push --activate`, `project delete` and `project load` [ask first](../connecting.md#usb). [`job`](./session.md) follows a save, load or copy. [`abort`](./session.md) stops a push.

```bash
s32 --loopback --yes project create internal Scratch --seed empty
s32 --loopback --yes project load internal 2
s32 --loopback project save --wait
s32 --loopback project save-as internal "Take 2"
s32 --loopback project save-as sd Gig --activate --wait
```

## Save As

`project save-as` leaves the original as it was last saved. Without `--activate` you stay on the original, unsaved changes and all. With `--activate` the new project becomes the one you are playing once it is stored.

## Push

A `push` without `--activate` only stores the file. The project you are playing doesn't change. If a push is stopped or fails, knobs and MIDI keep working, nothing new is stored, and the project you are playing stays as it was.

## Answers to expect

| Answer | When |
|--------|------|
| Busy | The card is still reading its project list (or just came out), a storage job is running, or a pull or push is still open. Try again |
| Not found | No card, or no project with that uid |
| No space | The volume is full |
| Not ready | The project is still loading |
| Storage error | The card failed to read. Take it out, put it back in, try again |

## Files

A `.s32p` file is a whole project. Keep it as a backup, restore it with `push`, or decode it with the layout from `s32 layout --emit-layout-json`.

## JSON

`project list`, `project active` and `storage` take [`--json`](./session.md#json-output). `project list --json` is `{"projects":[{"active":true,"vol":0,"uid":1,"gen":1,"storageVersion":257,"name":"Project 1"},...]}`; `storageVersion` is the number the text prints as `v257`. Every other project command, including `project delete` and `project load`, has no JSON form but takes [`--dry-run`](./session.md#preview-before-a-change): it names the stored project it would remove or replace and sends nothing.

```bash
s32 --loopback project delete internal 3 --dry-run
```

```text
dry-run: s32 project delete internal 3 (nothing sent)
  would permanently remove the stored project vol=0 uid=3 name=Project 3
```
