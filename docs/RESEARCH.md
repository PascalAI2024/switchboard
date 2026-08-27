# Research: the universal device hub

*Compiled 2026-08-27. Treat reported third-party numbers as approximate; primary sources
linked where they exist.*

## 1. Why vendor software exists at all

USB HID solves basic input without drivers: any keyboard types, any mouse moves, out of
the box. What HID does **not** standardize is everything vendors sell on: DPI stages,
polling rate, key remapping, macros, profile slots, lighting, battery telemetry.

Vendors implement those over **vendor-defined HID reports** — feature/output reports on a
vendor usage page whose byte layout is proprietary and undocumented. There is no API to
call. That is the entire reason each brand ships its own app, and the entire technical
problem a universal hub must solve: someone has to learn each device's byte protocol.

Two consequences shape everything downstream:

- **Every device model needs reverse engineering** (USB capture → protocol analysis),
  and vendors silently revise controllers within the same model name, so "supported"
  decays without maintenance.
- **Most gaming peripherals store settings in onboard flash.** The vendor app is only
  needed at configuration time; the resident service is almost always unnecessary.
  Logitech documents this explicitly as
  [On-Board Memory mode](https://support.logi.com/hc/en-001/articles/360057433714) —
  configure, save to device, uninstall.

## 2. The landscape — who has already built pieces of this

### OpenRGB — the proof that crowdsourced modules work
[openrgb.org](https://openrgb.org/) · [GitLab](https://gitlab.com/CalcProgrammer1/OpenRGB) · GPLv2

Open-source, cross-platform lighting control. The community reverse-engineers each
device's protocol and contributes a controller implementation; coverage is reported at
1,000+ devices across dozens of brands (motherboards, RAM, GPUs, peripherals, fans).
Reached 1.0 release-candidate status in 2026. Ships an **SDK server** (network protocol)
that third-party apps use to drive lighting — which is exactly how Switchboard should
consume it rather than re-implementing it.

Two lessons: crowdsourcing device protocols **works at scale**, and it is a permanent
maintenance treadmill. Also a structural lesson: contributions are C++ controller code,
which keeps the contribution bar high — a gap a declarative format can attack.

### SignalRGB — the commercial version
[signalrgb.com](https://signalrgb.com/)

Polished freemium hub for lighting; some manufacturers submit device plugins directly.
Full feature set is a paid subscription, and it is widely reported as heavier on
resources than OpenRGB. Validates that there is consumer demand for "one app instead of
ten," and that vendors will sometimes *cooperate* with a hub that has users.

### Windows Dynamic Lighting + HID LampArray — the OS admits the problem
[Microsoft Learn: LampArray](https://learn.microsoft.com/windows/uwp/devices-sensors/lighting-dynamic-lamparray) ·
[Microsoft Support](https://support.microsoft.com/en-us/windows/hardware/input-devices/control-dynamic-lighting-devices-in-windows)

Since Windows 11 23H2, devices implementing the open **HID LampArray** standard are
controlled from Settings with no vendor app. This is Microsoft standardizing the lighting
slice at the OS level. Limits: lighting only, and only devices whose firmware implements
LampArray — the installed base of proprietary hardware is excluded. Trendline worth
betting on: where a standard exists, defer to it.

### QMK / VIA / Vial — keyboards solved, if the firmware cooperates
[qmk.fm](https://qmk.fm/) · [caniusevia.com](https://caniusevia.com/) · [get.vial.today](https://get.vial.today/)

Open keyboard firmware plus universal configurators. Vial stores the keyboard's own
definition *on the device* and configures in real time; both have **browser-based
configurators over WebHID** — zero installed software. The catch: only works on keyboards
running QMK-family firmware, which mainstream vendor boards do not. Two lessons for
Switchboard: device-describes-itself is the ideal end state, and WebHID proves
config-without-install is viable.

### libratbag / Piper — the direct model, Linux-only
[github.com/libratbag/libratbag](https://github.com/libratbag/libratbag)

A vendor-neutral daemon exposing mouse/peripheral *settings* (DPI, buttons, profiles,
LEDs) over DBus, with per-device data files contributed by the community, and Piper as
the GUI. This is architecturally the closest thing to Switchboard that exists — **and it
has no Windows equivalent.** That absence is the market gap.

### DDC/CI — monitors already speak a standard
Every modern monitor supports DDC/CI (brightness, contrast, input source, volume over
the display cable). Vendor OSD apps (AOC G-Menu, LG OnScreen Control, etc.) are mostly
skins over it. Open tools — [Twinkle Tray](https://twinkletray.com/),
[Monitorian](https://github.com/emoacht/Monitorian), NirSoft ControlMyMonitor — already
replace them all. Switchboard should include a DDC/CI panel: cheap to build (standard
protocol, `ddc-hi` crate candidate), instantly useful, zero reverse engineering.

### Driverless printing — the end state to point at
IPP Everywhere / Mopria certified printers work with zero vendor software; Windows is
actively moving to driverless IPP printing. Print is out of scope for Switchboard
precisely because standards already won there — but it is the strongest argument that
the vendor-suite model is dying where standards exist.

## 3. Technical foundations a builder needs to know

- **Userspace HID access** is enough: hidapi (and WebHID in Chromium) can read/write
  feature and output reports without kernel drivers. Exception: some devices/interfaces
  are claimed exclusively by Windows (e.g. system keyboards' boot interface), which
  constrains what is reachable — a P0 verification item.
- **The RE workflow** is stable and well-trodden: run the vendor app inside a USB capture
  (Wireshark + USBPcap on Windows), change one setting at a time, diff the reports,
  identify the byte layout, replay without the vendor app.
- **Interference:** two apps driving one device fight each other. OpenRGB solves this
  socially (uninstall the vendor app); Switchboard inherits the same posture.
- **Churn:** the module registry needs device *revision* identity (VID/PID + firmware
  markers), not just model names.

## 4. Key takeaways

1. The idea is validated — twice (OpenRGB open, SignalRGB commercial) — but only for
   lighting. **Settings on Windows is unclaimed.**
2. Delegate anything a standard or a mature project already covers: lighting → OpenRGB
   SDK; LampArray devices → Windows; print → IPP. Differentiate on settings + DDC/CI +
   contributor experience.
3. Declarative modules (data files) beat code plugins for contribution bar and safety —
   libratbag proved the format, Vial proved the self-describing ideal.
4. The novel lever is **AI-assisted protocol drafting from USB captures** — it attacks
   the one bottleneck (skilled RE labor) that every predecessor accepted as fixed.
5. Onboard memory means the win condition for most users is "configure once, nothing
   resident" — the resident footprint of Switchboard itself must be near-zero, which is
   a stack decision, not a nice-to-have.
