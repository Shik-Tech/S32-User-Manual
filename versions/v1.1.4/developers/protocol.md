# USB protocol

The s32 CLI is the supported client for the S32 Editor USB interface. This page is the overview for anyone writing their own.

## Interface

A vendor USB interface: VID `0483`, PID `A56E`, bulk endpoints `0x05` (out) and `0x85` (in). One client at a time, with no password ([Connect](./connecting.md#usb)).

The protocol version is **4**. `hello` reports it, and a client that speaks another version is refused instead of misread. The s32 CLI version (`s32 --version`) is separate from the firmware version.

## Session

1. Claim the interface.
2. Send `hello`. Check that `protocolVersion` is 4.
3. Send commands. Each request is a CRC-checked frame. The reply carries a status and an optional payload.
4. Subscribe to events for live updates (`watch` in the s32 CLI).

Every reply carries the command and `seq` of the request it answers. Match both. A client stopped mid-transfer leaves unread replies behind, and the next client gets them first. The s32 CLI starts at a random `seq`, drops anything waiting before `hello`, and skips replies that don't match. A new `hello` ends any push or pull the last client left open.

Two statuses come from the rules, not from errors: `Forbidden` (that action is panel-only) and `TooMany` (changes sent faster than the S32 takes them; wait and retry).

## Discovering the device

```bash
s32 --usb hello
s32 --usb schema --emit-schema-json schema.json
s32 layout --emit-layout-json layout.json
```

`schema` lists device types, parameter names and enum tables (macro modes, menus). `layout` describes the `.s32p` project file for tools that decode it. It needs no S32.

`hello` also reports a capability mask. Bit 19 (`524288`) means the device enforces the panel's one-mapping rule: a mapping write from the editor replaces the parameter's other knob or MIDI mapping.

Bit 20 (`1048576`) means CLEAR RACK resets, and DUPLICATE RACK copies, the rack's BYPASS and IN SNAPSHOT.

Bit 21 (`2097152`) means the rack start rules: REMOVE never refuses the first device, a needed MIDI IN or CLOCK goes first, a top CLOCK stays on top under MOVE, a load keeps every device, and DUPLICATE RACK keeps the source order.

Bit 22 (`4194304`) means a Step Seq step changed on the S32 is reported as an event; re-read the pattern when it arrives.

Bit 23 (`8388608`) means `view nav` (and the editor's "show on the S32") can open a Step Seq or Euclid setting row, or focus a step or layer: `row=`, `step=` and `layer=` need this bit.

Names such as `mode=cc`, `clockMode=internal` and `view nav racks` are s32 CLI words. On the wire they are the integers in the schema's enum tables.
