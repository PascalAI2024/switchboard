# Architecture Decision Records

Format: context → decision → consequences. Reverse-chronological additions; never edit a
decided ADR, supersede it.

---

## ADR-001 — Tauri v2 over Electron / WinUI / raw Win32

**Context.** The product's entire thesis is that device software should not be a heavy
resident. Shipping the hub as a 300MB Electron app with its own updater would recreate
the disease. WinUI/.NET is off-stack for this team; raw Win32 makes UI iteration too
expensive.

**Decision.** Tauri v2: Rust binary + system webview, React/TS/Vite frontend.

**Consequences.** Small installer and memory footprint; the UI layer is the team's home
turf; the Rust side doubles as the core's process host. Cost: Tauri's Windows webview
(WebView2) is an Edge dependency — acceptable, it ships with Windows 11.

---

## ADR-002 — Rust core, not Node/Bun native HID bindings

**Context.** The core must (a) speak raw HID feature/output reports, (b) optionally run
as a tray resident or service with near-zero footprint, (c) never require a JS runtime
to be resident. `node-hid` exists but drags a runtime into residency.

**Decision.** Core in Rust. Candidate crates to verify in P0: `hidapi` (transport),
`windows-rs` (device notifications), `ddc-hi` (DDC/CI), `serde`/`schemars` (modules),
`specta`/Tauri IPC for typed TS bindings.

**Consequences.** Two-language repo (Rust core, TS everywhere else) — mitigated by
generated TS bindings and by keeping all tooling/scripts in Bun+TS. The team's Rust
surface is small and isolated to transport + engine.

---

## ADR-003 — Delegate lighting to OpenRGB; delegate LampArray to Windows

**Context.** OpenRGB has already crowdsourced lighting protocols for on the order of a
thousand devices under GPLv2, and exposes an SDK server protocol for third-party
control. Windows Dynamic Lighting handles HID LampArray devices at the OS level.
Re-implementing either is years of duplicated reverse engineering with zero
differentiation.

**Decision.** Switchboard never implements lighting protocols. It ships an OpenRGB SDK
*client* bridge (talks to a local OpenRGB server over its network protocol — no GPL
code linkage) and leaves LampArray devices to Windows Settings.

**Consequences.** Lighting users need OpenRGB installed — an acceptable dependency on
the best tool in class. Switchboard's scope stays sharply "settings + monitors +
contributor funnel," which is the unclaimed gap.

---

## ADR-004 — Declarative JSON modules, not code plugins

**Context.** OpenRGB's contribution unit is C++ controller code: high contribution bar,
requires trusting contributed code. libratbag and Vial demonstrate data-driven device
definitions. The AI-assisted capture workflow can only ever emit *data* safely.

**Decision.** Device support is a schema-validated JSON file interpreted by the engine.
No scripting in v0; protocols the format can't express are spec feedback
([MODULES.md](MODULES.md)).

**Consequences.** Low contribution bar, machine-verifiable PRs (schema + capture
replay), no arbitrary-code risk. Cost: genuinely gnarly protocols (encryption, complex
handshakes) may be inexpressible — those devices wait for a deliberate spec revision
rather than an escape hatch.

---

## ADR-005 — MIT license now; revisit before external contributions arrive

**Context.** A public repo with no license is proprietary by default, which contradicts
the project's premise. MIT is the team default and frictionless. However: for a project
whose value concentrates in a crowdsourced registry, copyleft (GPL, like OpenRGB) is a
defensible strategy against proprietary strip-mining of the module registry.

**Decision.** MIT today. **Hard checkpoint:** re-decide (MIT vs GPL vs split licensing —
permissive engine, copyleft registry) before merging the first external module
contribution, because relicensing gets harder with every outside contributor.

**Consequences.** Zero friction now; a dated obligation on the roadmap (tracked in
PLAN.md P5).

---

## ADR-006 — Windows-first, userspace-only

**Context.** Linux already has libratbag; macOS is a different HID sandbox regime; the
gap is Windows. Kernel drivers would grant access to devices Windows claims exclusively,
at the cost of signing, security surface, and trust — the opposite of this project's
posture.

**Decision.** Windows is the only v1 target; all device access is userspace (hidapi /
WebView-adjacent APIs). Devices unreachable from userspace are documented out of scope.
Keep the core portable (hidapi is cross-platform) but spend no v1 effort on other OSes.

**Consequences.** Some devices (notably system keyboard boot interfaces) may be
partially or fully unreachable — P0 must verify reachability against real hardware
early, so scope claims stay honest.
