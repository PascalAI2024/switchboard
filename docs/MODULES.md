# Device modules — draft spec v0

A module is **one JSON file describing one device's settings protocol**. The engine
interprets it; contributors never ship code. This draft exists to be torn apart in P1 —
it records intent, not a frozen format.

## Design rules

1. **Capability-oriented.** A module declares which of a fixed capability vocabulary the
   device supports (`dpi`, `polling_rate`, `keymap`, `macro`, `profile_slot`,
   `battery`, …). The UI is generated from capabilities — no per-device UI code.
2. **Byte templates, not scripts.** Reports are declared as hex templates with typed
   placeholders. No expressions, no loops in v0 — if a protocol can't be expressed,
   that's spec feedback, not a reason to add a scripting engine. (Known open question:
   checksums and multi-step handshakes; see below.)
3. **Identity is revision-aware.** Match on VID/PID plus optional firmware/interface
   markers, because vendors silently revise controllers within one model name.
4. **Evidence travels with the module.** A module PR includes the capture it was
   verified against (payloads only, serials stripped), so CI can replay-check it and
   future maintainers can re-derive intent.

## Sketch

```jsonc
{
  "$schema": "https://raw.githubusercontent.com/PascalAI2024/switchboard/main/schema/module.v0.json",
  "id": "examplecorp-gk100",
  "name": "ExampleCorp GK100 Keyboard",
  "match": {
    "vendor_id": "0x1234",
    "product_id": "0xABCD",
    "interface": 1,
    "usage_page": "0xFF00"          // vendor-defined page carrying the config reports
  },
  "capabilities": {
    "polling_rate": {
      "values": [125, 250, 500, 1000],
      "write": {
        "report_type": "feature",
        "template": "0A 01 {value:enum_index:u8} 00 00 00 00 00"
      },
      "read": {
        "report_type": "feature",
        "request": "0A 02 00 00 00 00 00 00",
        "parse": { "offset": 2, "type": "enum_index:u8" }
      }
    },
    "profile_slot": {
      "slots": 3,
      "write": {
        "report_type": "output",
        "template": "0B {value:u8} 00 00 00 00 00 00"
      }
    }
  },
  "quirks": {
    "settle_ms": 50                  // delay between consecutive writes
  },
  "provenance": {
    "captured_with": "vendor app v2.0 on Windows 11",
    "capture": "captures/examplecorp-gk100.json",
    "verified": "replay"             // replay | on-device | unverified
  }
}
```

## Registry layout

```
modules/
  <vendor-slug>/
    <device-slug>.json
captures/                # payload-only evidence, serials stripped
schema/module.v0.json    # JSON Schema; CI validates every module against it
```

Contribution flow: `tools/capture` drafts the module → PR → CI runs schema validation +
capture replay → human review (a maintainer sanity-checks the protocol claims) → merge.

## Open questions for P1

- **Checksums / CRCs**: declare as a named algorithm + byte range
  (`"checksum": {"algo": "sum8", "range": [0, 6], "at": 7}`) — is a small fixed
  algorithm vocabulary enough for real devices?
- **Handshakes**: some devices need an init sequence before accepting writes. Ordered
  `sequence` arrays per capability, or a module-level `session` block?
- **Reading current state**: many cheap devices are write-only (state lives in the
  vendor app, not the device). The UI must treat "unknown current value" as a
  first-class state, not an error.
- **Keymap capability**: full remap tables are an order of magnitude more complex than
  scalar settings — probably its own spec revision (v0 ships scalars + profile slots
  first).
- **Versioning**: modules carry `$schema`; the engine refuses schemas newer than it
  understands. Migration story TBD.
