# USB protocol

The s32 CLI is the supported client for the S32 Editor USB interface. This page is the overview for anyone writing their own.

## Interface

A vendor USB interface: VID `0483`, PID `A56E`, bulk endpoints `0x05` (out) and `0x85` (in). One client at a time, with no password ([Connect](./connecting.md#usb)).

The protocol version is **5**. `hello` reports the protocol of the session, and a client that speaks a version the firmware does not serve is refused instead of misread: firmware 1.1.5 serves **4 and 5**, older firmware serves one of them. Offer 5 first; if the refusal names 4, send one more `hello` with 4 and keep using it. The s32 CLI version (`s32 --version`) is separate from the firmware version.

Protocol 5 implies every capability bit of protocol 4: a 1.1.5 `hello` reports only the new protocol-5 bits in its capability mask and the runtime facts in a `presence` byte instead (bit 0 microSD card; bit 1 is reserved). On protocol 4 the mask still names each feature, exactly as in 1.1.4.

| Protocol-5 bit | Mask | Feature | Named line | Firmware |
|---|---|---|---|---|
| 0 | `0x00000001` | Opening a Step Seq step's parameter with that parameter highlighted (`view nav … step=N field=NAME`) | `stepFieldNav yes` | 1.1.5 |

## Session

1. Claim the interface.
2. Send `hello`. Check that `protocolVersion` is 4 or 5 (a refusal names the protocol to use).
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

`hello` also reports the feature lines and, for programs, the capability mask of its protocol. Protocol 5 implies all of the protocol-4 features, so its mask holds only the protocol-5 bits above and the named lines carry the rest; on protocol 4 each is a bit. Follow the named lines (`rackStart yes`, `navTarget yes`, `stepFieldNav yes`, ...) or the JSON keys rather than the raw mask. `stepFieldNav` is always `no` on protocol 4.

Protocol 5 implies the one-mapping rule: a mapping write from the editor replaces the parameter's other knob or MIDI mapping (protocol 4, bit 19 `524288`).

It implies CLEAR RACK resets, and DUPLICATE RACK copies, the rack's BYPASS and IN SNAPSHOT (protocol 4, bit 20 `1048576`).

It implies the rack start rules: REMOVE never refuses the first device, a needed MIDI IN or CLOCK goes first, a top CLOCK stays on top under MOVE, a load keeps every device, and DUPLICATE RACK keeps the source order (protocol 4, bit 21 `2097152`).

It implies a Step Seq step changed on the S32 is reported as an event; re-read the pattern when it arrives (protocol 4, bit 22 `4194304`).

It implies `view nav` (and the editor's "show on the S32") can open a Step Seq or Euclid setting row, or focus a step or layer: `row=`, `step=` and `layer=` (protocol 4, bit 23 `8388608`).

Names such as `mode=cc`, `clockMode=internal` and `view nav racks` are s32 CLI words. On the wire they are the integers in the schema's enum tables.
