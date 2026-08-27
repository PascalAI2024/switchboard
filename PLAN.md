# Switchboard — build plan

Phases are sequential; each has concrete acceptance criteria. Nothing below P0 exists
yet — this repo is research + plan.

## P0 — Scaffold & feasibility spikes

Goal: prove the stack and the risky assumptions before building on them.

| # | Task | Acceptance criteria |
|---|---|---|
| 0.1 | Tauri v2 scaffold (React+TS+Vite, Bun scripts, Oxlint/Oxfmt, clippy/rustfmt, GH Actions CI) | `bun run tauri dev` opens a window; CI runs lint+typecheck+`cargo check` green on PR |
| 0.2 | HID enumeration spike | App lists connected HID devices with VID/PID/usage-page; verified against ≥3 real devices |
| 0.3 | **Reachability spike** (the honest-scope gate) | Documented matrix: for each test device, which interfaces/report types are writable from userspace on Windows; includes at least one keyboard, one mouse |
| 0.4 | Crate verification | `hidapi`, `ddc-hi`, `windows-rs` confirmed (or replacements chosen) with a one-paragraph note each in DECISIONS.md |
| 0.5 | Read/write one real vendor feature report | One setting on one real device changed from our code, no vendor app installed |

## P1 — Module engine v0

Goal: the declarative format works end-to-end on real hardware.

| # | Task | Acceptance criteria |
|---|---|---|
| 1.1 | Finalize module schema v0 from MODULES.md draft | `schema/module.v0.json` exists; CI validates all modules; open questions in MODULES.md resolved or explicitly deferred |
| 1.2 | Engine interprets modules | Given a module file, engine renders byte-exact reports (unit-tested against fixtures) |
| 1.3 | Two reference modules, hand-written | Two real devices fully configurable through the UI via their modules only |
| 1.4 | Capability-driven UI | UI renders controls from module capabilities; zero per-device UI code |
| 1.5 | Dry-run / replay mode | Engine can diff its rendered reports against a recorded capture and report byte-level mismatches |

## P2 — Monitor control (DDC/CI)

Goal: immediate everyday value with zero reverse engineering.

| # | Task | Acceptance criteria |
|---|---|---|
| 2.1 | DDC/CI panel | Brightness/contrast/input-source control works on ≥2 monitor brands |
| 2.2 | Multi-monitor + hotkeys | Per-monitor and all-monitor brightness; global hotkey support behind the opt-in resident |
| 2.3 | Resident footprint gate | Tray resident measured; **fails the phase if >25MB RAM or measurable idle CPU** |

## P3 — Capture & AI-assist pipeline (the risk-retirement phase)

Goal: answer whether the novel bet works, with real captures, before scaling anything.

| # | Task | Acceptance criteria |
|---|---|---|
| 3.1 | `tools/capture` CLI | Guided capture (Wireshark/USBPcap wrapper) produces labeled, payload-only, serial-stripped capture files |
| 3.2 | Diff engine | Per-label report diffs isolate changed bytes across ≥3 real device captures |
| 3.3 | LLM draft pass | For ≥3 devices: drafted module passes replay verification for scalar settings with ≤1 human correction each — **if this fails, write the honest post-mortem and re-scope P3 before proceeding** |
| 3.4 | PR automation | `capture submit` opens a registry PR with module + evidence; CI replay-checks it |

## P4 — Bridges

| # | Task | Acceptance criteria |
|---|---|---|
| 4.1 | OpenRGB SDK client | With a local OpenRGB server running, Switchboard lists its devices and applies a color/profile via the SDK protocol |
| 4.2 | Graceful absence | Without OpenRGB installed, lighting UI shows a one-click "install OpenRGB" pointer — no errors, no nagging |

## P5 — Community launch

| # | Task | Acceptance criteria |
|---|---|---|
| 5.1 | **License checkpoint (ADR-005)** | MIT-vs-GPL-vs-split re-decided and recorded BEFORE first external module PR is merged |
| 5.2 | Contributor docs | A stranger with a supported-adjacent device can go capture→PR from docs alone (tested on one real outsider) |
| 5.3 | Module registry hygiene | CODEOWNERS, PR template with evidence checklist, replay CI required |
| 5.4 | v0.1 release | Signed installer via GH Releases; README quickstart verified on a clean Windows 11 VM |

## Open decisions (owner: Pascal)

| Decision | Needed by | Notes |
|---|---|---|
| Final product name | before v0.1 release | "Switchboard" is a working codename |
| License final call | P5.1 | ADR-005 |
| First reference devices for P0/P1 | P0.2 | Pick from hardware actually on hand |
