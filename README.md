# AppleMonitorCenter

<p align="center">
  <img src="AppleMonitorCenter/Assets.xcassets/AppIcon.appiconset/icon_256.png" width="128" alt="AppleMonitorCenter icon">
</p>

<p align="center">
  <a href="https://github.com/LorysMay/AppleMonitorCenter/releases"><img src="https://img.shields.io/github/v/release/LorysMay/AppleMonitorCenter?label=release&color=4c8dff" alt="Latest release"></a>
  <a href="https://github.com/LorysMay/AppleMonitorCenter/releases"><img src="https://img.shields.io/github/downloads/LorysMay/AppleMonitorCenter/total?color=4c8dff" alt="Downloads"></a>
  <a href="#supported-hardware--macos"><img src="https://img.shields.io/badge/macOS-13%2B%20Apple%20Silicon%2FIntel-black" alt="macOS 13 or later, Apple Silicon or Intel"></a>
  <a href="LICENSE"><img src="https://img.shields.io/badge/license-Proprietary-blue" alt="License: Proprietary"></a>
</p>

AppleMonitorCenter is a native macOS app for monitoring your Mac — **Apple Silicon and Intel** alike (universal binary). It shows real-time CPU/GPU usage, temperatures, clock speeds, power draw, memory, storage, battery and fan data, with an adaptive Dashboard and a compact menu bar panel.

> **This repository ships compiled releases only — no source code.** You'll find the app binary under [Releases](../../releases) and app screenshots below. Bug reports and feature requests are welcome via [Issues](../../issues).

## Screenshots

*(see the repository's screenshot files for the Dashboard, Sensors view and menu bar panel)*

## Supported hardware & macOS

- **CPU:** any Mac with Apple Silicon (M1 through M5, including Pro/Max/Ultra variants) **or** Intel — the app is a universal binary (arm64 + x86_64).
- **macOS:** 13 Ventura or later.
- On Apple Silicon, readings come from IOReport and the SMC keys for that specific SoC generation.
- On Intel Macs, the same kind of data is read from the "classic" SMC keys, `sysctl`, and — when installed — Intel Power Gadget.
- Integrated GPU, discrete AMD GPUs and Thunderbolt eGPUs are all read when the driver exposes usage, temperature, clock, power, voltage and fan data.

## Features

- CPU and GPU monitoring: usage, temperature, clock speed, power draw and history graphs.
- Per-core CPU usage, split between Efficiency and Performance cores.
- Unified memory status and memory pressure.
- SMC sensors grouped by category (temperatures, voltages, currents).
- Mac hardware info, installed RAM and macOS version, with the official Mac model picture pulled from macOS system resources.
- SSD, battery and fan monitoring when present.
- Automatic, manual and custom-curve fan control.
- Configurable menu bar panel (temperature, RPM, usage, battery %, and more — up to 10 independently configurable sections).
- Responsive Dashboard that adapts to different MacBook screen sizes/resolutions.
- Interface localized in multiple languages.

## Installation

1. Download the latest `.app`/`.zip` (or `.dmg`, if provided) from [Releases](../../releases).
2. Move `AppleMonitorCenter.app` to `/Applications`.
3. Open the app — it's signed with a Developer ID and notarized by Apple, so it opens normally with a plain double-click, no Gatekeeper warning or right-click workaround needed.
4. When macOS asks, approve the privileged helper used for fan control in **System Settings → General → Login Items & Extensions**.

Fan control writes are delegated to a small privileged helper registered via `SMAppService`; the main app always runs with your normal user privileges. See [Fan control safety](#fan-control-safety) below.

## Fan control safety

- The main app runs with your normal user privileges. Only the SMC writes needed for fan control are delegated to a separate helper running as `root`.
- Communication happens over XPC, and the helper verifies the client's code signature.
- RPM values are clamped to the min/max range published by the firmware.
- Fans are returned to automatic mode before the helper is removed.
- The firmware's own thermal protections stay active regardless of the manual curve you set.

Manual fan control changes your Mac's thermal behavior. Avoid setting speeds too low under load, and use curves consistent with the temperatures you're seeing.

## Reporting problems / opening an Issue

Since SMC, IOReport and the per-chip DVFS tables are undocumented by Apple and differ between Mac models, the fastest way to fix a "sensor X isn't showing" or "value Y looks wrong" report is to attach one (or both) of the files below to your Issue, together with:
- your Mac model (e.g. "MacBook Pro M5 14\" 2025") and macOS version,
- whether you're on Intel or Apple Silicon,
- a short description of what's wrong and, if relevant, a screenshot.

Neither file contains personal data — only your Mac model, macOS version and hardware sensor values.

### Diagnostic report

**Settings → General → Diagnostics → "Generate diagnostic report"**

Writes `AppleMonitorCenter-Diagnostics-<date>.txt` to your Desktop. This is the most complete report: SMC connection status and full key list, recognized sensors with their source keys, raw SMC temperature/voltage/current keys, per-core mapping coverage (how many Efficiency/Performance/GPU cores are actually mapped vs. what the CPU actually has), IOReport energy and frequency (DVFS) channels, GPU driver metrics, fan status, and privileged helper status. This is generally the file to attach for **anything** that isn't reading correctly.

### Sensor dump

**Settings → Sensors → "Export sensor dump"**
*(on Apple Silicon this button lives inside the "Verify sensors" section — see below)*

Writes `sensor-dump-<Mac model>-<timestamp>.txt` to your Desktop: every raw SMC key with its raw bytes, plus every HID sensor with the name macOS itself assigns it. This is what lets a **new, not-yet-supported chip generation** get proper sensor names added — if you're on hardware AppleMonitorCenter doesn't fully recognize yet, this is the file to send.

### Verify sensors (Apple Silicon only)

**Settings → Sensors → "Verify sensors" → "Run verification (~6 min)"**

Available only on Apple Silicon Macs (the per-core SMC key tables are chip-generation specific; Intel cores use a different, non-ambiguous key scheme, so there's nothing to verify there). This runs the CPU through three known load states and checks that the keys currently mapped to each core actually behave like real per-core sensors. It applies the result per cluster: if it resolves Efficiency cores but stays ambiguous on Performance (or vice versa), only the resolved cluster is updated — the rest keeps the existing verified table. Your Mac will get warmer and fans will spin up during the test; that's expected. If the result doesn't match what you're seeing in the Dashboard, export a sensor dump as well and attach both to your Issue.

## License

This is proprietary software: no source code is published, so an open-source license (MIT, GPL, Apache…) wouldn't really apply — those govern rights over source you can read and modify, which isn't the case here. See [`LICENSE`](LICENSE) for the exact terms; in short: the compiled app is free to download and use, personally or professionally, but you may not redistribute it (repackaged or under another name), reverse-engineer/decompile it, or reuse its assets (icon, code, screenshots) without permission. All rights reserved © LorysMay.
