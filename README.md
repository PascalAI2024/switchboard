# Switchboard

**One open hub for every device. Kill the vendor bloatware.**

A typical Windows gaming/workstation PC accumulates a dozen vendor apps — each with its
own resident service, tray icon, and auto-updater — to do four basic jobs: configure a
keyboard, set mouse DPI, dim a monitor, sync some lighting. Hundreds of megabytes of RAM
and a dozen background processes for settings that mostly live *in the device firmware
anyway*.

Switchboard is an open-source Windows hub that replaces them: a tiny native core, a
modern UI, and **crowdsourced, declarative device modules** — support for a new device is
a data file the community contributes, not a C++ plugin.

> Status: **research phase**. This repo currently holds the research, architecture
> proposal, and build plan. "Switchboard" is a working codename.

## The one decision that shapes everything

**We do not rebuild lighting control.** [OpenRGB](https://openrgb.org/) has already
crowdsourced reverse-engineered lighting protocols for on the order of a thousand
devices; competing with that is wasted life. Switchboard targets the gap nobody owns on
Windows — **device *settings*** (remapping, DPI, polling, profiles, onboard memory) plus
**monitor control** (DDC/CI) — and bridges to OpenRGB's SDK for lighting. See
[docs/DECISIONS.md](docs/DECISIONS.md).

Linux has this in [libratbag/Piper](https://github.com/libratbag/libratbag) — a settings
daemon with crowdsourced device definition files. **Windows has no open equivalent.**
That's the niche.

## What makes it plausibly scale

The historical bottleneck for crowdsourced device support is that every contribution
requires skilled USB reverse engineering. Switchboard's bet is an **AI-assisted capture
workflow**: record USB traffic while using the vendor app, and tooling drafts the device
module from the capture diff. The contributor's job collapses to "wiggle the slider while
recording." Details in [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md).

## Stack

| Layer | Choice | Why |
|---|---|---|
| Desktop shell | **Tauri v2** | A hub that replaces bloatware cannot itself be a 300MB Electron resident. Tauri ships a small binary with a webview UI. |
| UI | **React + TypeScript + Vite** | Team standard. |
| Core / daemon | **Rust** (`hidapi`, `windows-rs`, `ddc-hi` — candidates, verify in P0) | Raw HID access, a tray/service resident with a tiny footprint, no runtime. |
| Device modules | **Declarative JSON** + JSON Schema | Data, not code. Low contribution bar, no arbitrary code execution from the crowd. |
| Tooling / scripts | **Bun + TypeScript** | Team standard. Capture CLI, module validation, registry checks. |
| Lint / format | Oxlint + Oxfmt (TS), clippy + rustfmt (Rust) | Team standard. |
| CI | GitHub Actions | Module schema validation on every PR. |

## Repo map

| File | What's in it |
|---|---|
| [docs/RESEARCH.md](docs/RESEARCH.md) | The full landscape: what exists, why it exists, where the gap is |
| [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) | Components, data flow, the AI-assisted RE workflow |
| [docs/MODULES.md](docs/MODULES.md) | Draft v0 spec for declarative device modules |
| [docs/DECISIONS.md](docs/DECISIONS.md) | ADRs — stack, scope, and license decisions with rationale |
| [PLAN.md](PLAN.md) | Phased build plan with acceptance criteria. P0 is the scaffold. |

## License

MIT — see [LICENSE](LICENSE). Revisit before external module contributions arrive
([ADR-005](docs/DECISIONS.md)).
