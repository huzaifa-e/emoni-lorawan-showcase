<p align="center">
  <img src="media/icon.svg" width="88" alt="EMONI logo">
</p>

<h1 align="center">EMONI LoRaWAN device and fleet platform</h1>

<p align="center">
  <em>I built a battery-powered LoRaWAN meter reader, and the platform that keeps a fleet of them healthy: from the uplink in a basement to a fix rolled out over the air, with hardware and AI in the loop.</em>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/device-hardware%20%2B%20firmware-E7352C" alt="Device: hardware and firmware">
  <img src="https://img.shields.io/badge/radio-LoRaWAN%20EU868%20%C2%B7%20Class%20A-21D55C" alt="LoRaWAN">
  <img src="https://img.shields.io/badge/firmware-C%20%2F%20FreeRTOS%20%C2%B7%20on--device%20ML-E7352C" alt="Firmware">
  <img src="https://img.shields.io/badge/platform-TypeScript%20%C2%B7%20Next.js%20%C2%B7%20Express-3178C6" alt="TypeScript">
  <img src="https://img.shields.io/badge/data-SQLite%20%C2%B7%20SSE-0F80CC" alt="SQLite">
  <img src="https://img.shields.io/badge/bench-PPK2%20%C2%B7%20Python%20%C2%B7%20Tailscale-F59E0B" alt="Bench">
  <img src="https://img.shields.io/badge/AI-MCP%20server%20%C2%B7%20agent%20debugger-8A2BE2" alt="AI">
  <img src="https://img.shields.io/badge/UML-LaTeX%20%2F%20TikZ-008080" alt="LaTeX">
</p>

> This repository documents what I built and how I work, with screenshots of the real system, diagrams drawn in LaTeX, and a video. Source code, schematics, part numbers, payload formats and customer data stay private. Facts and drawings of the device come only from EMONI's public documents ([emoni.cloud/downloads](https://emoni.cloud/downloads/)).

<p align="center">
  <img src="media/overview.gif" width="900" alt="A tour of the platform: the fleet, a device page, a connectivity test, the remote lab, a sleep-current fix">
</p>

<p align="center">
  <b><a href="media/emoni-lorawan-showcase.mp4">Watch the full walkthrough</a></b> · 5 min · MP4 · 1920 × 1200
</p>

---

## The device

**EMONI** is a retrofit meter reader for gas, water and electricity meters, made by ISIS IC. **I designed and built its LoRaWAN device**: the hardware, the firmware, the on-device recognition, the LoRaWAN behaviour and the low-power design. It sits over the counter of an existing meter, photographs the digits, recognises them on the device and sends the absolute reading over LoRaWAN, on four AA cells for years.

<p align="center">
  <img src="media/device-strip.png" width="820" alt="The EMONI LoRaWAN device: front with its antenna, back with the hinged holder, and the controls (status LEDs, user button, reboot button)">
</p>
<p align="center"><sub>Drawings from the public EMONI datasheet.</sub></p>

<p align="center">
  <img src="media/lorawan-city.jpg" width="820" alt="LoRaWAN across a city: EMONI devices in basements send readings to rooftop gateways, which forward them to the network server and the application">
</p>

Devices in cellars and meter shafts are hard to diagnose remotely. A reading arrives late, and nobody knows whether the radio, the gateway, the network server or the firmware caused it. A battery drains faster than the budget because of a few microamps somewhere. A fix has to reach the device over a link that carries a few dozen bytes per downlink. So I built **a system around the device**: it tracks every device, gives me the tools to debug it, and lets me fix it remotely and verify the fix.

## What I built

| | |
|---|---|
| <img src="media/icons/device.png" width="40" alt=""> **The device** | hardware, firmware, on-device digit recognition, LoRaWAN join, retry and rejoin logic, deep sleep, calibration over local Wi-Fi, FUOTA with rollback |
| <img src="media/icons/tracking.png" width="40" alt=""> **A fleet dashboard** | lorawan.emoni.cloud: ingests every uplink from The Things Stack, ChirpStack, generic LoRaWAN and NB-IoT webhooks, and shows each device live: reading, battery and runtime left, signal, link health, consumption, anomalies |
| <img src="media/icons/debugging.png" width="40" alt=""> **Debugging tools** | a failure log that explains every late uplink, link health per gateway, a payload explorer, power tests with waveforms, connectivity tests that break the network on purpose, FUOTA tracking |
| <img src="media/icons/fixing.png" width="40" alt=""> **Remote fixes** | downlinks for interval and re-baselining, automatic re-baselining of implausible jumps, firmware rollouts over the air, all logged and confirmed |
| <img src="media/icons/remote-lab.png" width="40" alt=""> **A remote lab** | a bench host with a power profiler, console logging, flashing and test runners, reachable from anywhere over Tailscale, pushing every result to the dashboard |
| <img src="media/icons/ai.png" width="40" alt=""> **AI in the loop** | an MCP server with 26 tools over the whole fleet, and a debugger built into the dashboard that runs an AI agent next to the data, the firmware source and its history |

<p align="center">
  <img src="diagrams/architecture.svg" width="100%" alt="Architecture: devices and gateways in the field, network servers and the OTA cloud, the platform with ingest, API, database, dashboard, MCP endpoint and AI debugger, the bench with host, power profiler and Tailscale, and the clients">
</p>

---

## In pictures

<table>
  <tr>
    <td width="50%"><a href="tracking/README.md"><img src="tracking/images/fleet.jpg" alt="The fleet"></a><br><sub><b>Track</b> · every device live: reading, battery and runtime left, signal, payloads, FUOTA progress</sub></td>
    <td width="50%"><a href="tracking/README.md"><img src="tracking/images/device.gif" alt="A device page"></a><br><sub><b>Track</b> · one device on one page, from consumption heatmap to payload explorer</sub></td>
  </tr>
  <tr>
    <td><a href="debugging/README.md#failure-log"><img src="debugging/images/failure-log.jpg" alt="Failure log"></a><br><sub><b>Debug</b> · every late uplink with its cause, worked out from the data</sub></td>
    <td><a href="debugging/README.md#link-health"><img src="debugging/images/link-health.jpg" alt="Link health"></a><br><sub><b>Debug</b> · link health: ACKs, link margin, rejoins, delivery, SNR per gateway</sub></td>
  </tr>
  <tr>
    <td><a href="debugging/README.md#connectivity-tests"><img src="debugging/images/conn.gif" alt="Connectivity test"></a><br><sub><b>Debug</b> · connectivity tests: the network changed under a live device on purpose</sub></td>
    <td><a href="debugging/README.md#power-tests"><img src="debugging/images/power-run.jpg" alt="Power test"></a><br><sub><b>Debug</b> · power tests from the remote lab: sleep floor, every wake, battery life</sub></td>
  </tr>
  <tr>
    <td><a href="remote-lab/README.md"><img src="remote-lab/images/rig.gif" alt="The remote lab"></a><br><sub><b>Remote lab</b> · flash, power, measure and control a bench device from anywhere</sub></td>
    <td><a href="ai/README.md"><img src="ai/images/session.gif" alt="AI agent session"></a><br><sub><b>AI in the loop</b> · an agent driving the bench over SSH, condensed from a real session</sub></td>
  </tr>
  <tr>
    <td><a href="case-studies/README.md"><img src="case-studies/images/timeline.gif" alt="Sleep current over three days"></a><br><sub><b>Case study</b> · deep sleep 1840 → 28 µA in three days, every fix in firmware</sub></td>
    <td><a href="fixing/README.md#firmware-over-the-air"><img src="fixing/images/fuota-run.jpg" alt="FUOTA run"></a><br><sub><b>Fix</b> · a firmware update over LoRaWAN, chunk by chunk, verified on the device</sub></td>
  </tr>
</table>

## Diagrams, in LaTeX

<table>
  <tr>
    <td width="33%" align="center"><a href="device/README.md#one-wake-up"><img src="diagrams/act-wake-cycle.svg" alt="Device wake cycle"></a><br><sub>Activity · one wake-up of the device</sub></td>
    <td width="34%" align="center"><a href="platform/README.md#how-an-uplink-becomes-a-live-chart"><img src="diagrams/seq-uplink.svg" alt="Uplink sequence"></a><br><sub>Sequence · an uplink becomes a live chart</sub></td>
    <td width="33%" align="center"><a href="remote-lab/README.md#flash-power-measure-control-from-anywhere"><img src="diagrams/seq-remote-bench.svg" alt="Remote bench sequence"></a><br><sub>Sequence · remote flashing and measuring</sub></td>
  </tr>
  <tr>
    <td align="center"><a href="fixing/README.md#2--automatic-re-baselining"><img src="diagrams/act-auto-recal.svg" alt="Automatic re-baselining"></a><br><sub>Activity · automatic re-baselining</sub></td>
    <td align="center"><a href="fixing/README.md#the-whole-loop"><img src="diagrams/act-debug-loop.svg" alt="Track, debug, fix loop"></a><br><sub>Activity · track, debug, fix, verify</sub></td>
    <td align="center"><a href="debugging/README.md#the-ai-debugger"><img src="diagrams/seq-ai-debugger.svg" alt="AI debugger sequence"></a><br><sub>Sequence · the built-in AI debugger</sub></td>
  </tr>
</table>

All eleven diagrams, with their TikZ sources: **[diagrams](diagrams/README.md)**.

---

## Sections

| | Section | What it shows |
|---|---|---|
| <img src="media/icons/device.png" width="32" alt=""> | **[The device](device/README.md)** | what EMONI does, the drawings, and how one wake-up runs (activity diagram) |
| <img src="media/icons/platform.png" width="32" alt=""> | **[The platform](platform/README.md)** | architecture, how an uplink becomes a live chart (sequence diagram), stack, deployment, security |
| <img src="media/icons/tracking.png" width="32" alt=""> | **[Tracking](tracking/README.md)** | the fleet view and the device page: battery cycle and runtime, consumption, signal, KPIs |
| <img src="media/icons/debugging.png" width="32" alt=""> | **[Debugging](debugging/README.md)** | failure log, link health, payload explorer, connectivity tests, power tests, the AI debugger |
| <img src="media/icons/fixing.png" width="32" alt=""> | **[Fixing](fixing/README.md)** | downlinks, automatic re-baselining, FUOTA, and the whole track–debug–fix loop |
| <img src="media/icons/remote-lab.png" width="32" alt=""> | **[The remote lab](remote-lab/README.md)** | flashing, powering and measuring bench devices from anywhere: Tailscale, the bench host, the PPK2 |
| <img src="media/icons/ai.png" width="32" alt=""> | **[AI in the loop](ai/README.md)** | how an AI agent drives the bench and the dashboard, and what it may and may not do |
| <img src="media/icons/case-studies.png" width="32" alt=""> | **[Case studies](case-studies/README.md)** | a 65× lower sleep current, battery artefacts told apart from real drain, a network outage on purpose |
| <img src="media/icons/diagrams.png" width="32" alt=""> | **[Diagrams](diagrams/README.md)** | every sequence and activity diagram as LaTeX/TikZ source, PDF and SVG |

## What this shows about how I work

| Skill | Where to see it |
|---|---|
| **Building a product end to end**: hardware, firmware, cloud and tools, one person owning the whole chain | [Device](device/README.md), [platform](platform/README.md) |
| **Embedded and low-power debugging**: hypotheses, test builds, measurement before conclusion | [Case studies](case-studies/README.md#1--sleep-current-1840--28-µa), [remote lab](remote-lab/README.md) |
| **LoRaWAN in depth**: class A timing, confirmed uplinks, ADR, rejoin behaviour, downlink budgets, FUOTA | [Device](device/README.md), [connectivity tests](debugging/README.md#connectivity-tests), [FUOTA](fixing/README.md#firmware-over-the-air) |
| **Full-stack product engineering**: webhooks, data model, live UI, charts that answer real questions | [Platform](platform/README.md), [tracking](tracking/README.md) |
| **Test automation and hardware in the loop**: runners, sync services, reproducible runs | [Remote lab](remote-lab/README.md), [debugging](debugging/README.md#power-tests) |
| **AI tooling with guardrails**: MCP design, read-only by default, a human confirms every device change | [AI in the loop](ai/README.md) |
| **Security and operations**: no inbound ports, tokens and OAuth, secrets never leave the server, signed firmware | [Platform](platform/README.md#security) |
| **Documentation**: UML in LaTeX, public datasheet and technical reference | [Diagrams](diagrams/README.md) |

## At a glance

| | |
|---|---|
| **Device** | camera-based meter reader, on-device recognition, LoRaWAN EU868 Class A with OTAA, upload interval 1 min to 72 h, FUOTA with automatic rollback, ≈ 6 years on AA cells at the default 3 h interval (public datasheet) |
| **Platform** | Next.js 16 + React 19, Express + TypeScript, SQLite, server-sent events, Docker Compose, Cloudflare tunnel |
| **Bench** | Ubuntu bench host, Nordic PPK2 power profiler, console logging, Tailscale; power, connectivity and FUOTA runs pushed to the dashboard by three sync services |
| **AI** | MCP endpoint with 26 tools (OAuth or token), a built-in AI debugger that streams its trace and saves reports and notes |
| **Results** | deep sleep 197.5 → 28.3 µA on the bench (1840 µA on the first clean run), wake charge −20 %, every fix in firmware; network behaviour verified against the published rejoin rules |

## What is not in this repository

To respect my employer's IP and customer confidentiality, this repository contains **no** source code, schematics, part numbers, pin assignments, payload byte layouts, keys, internal addresses or customer data. Device names, identifiers and customer sites are masked or renamed in every screenshot. Where a product fact or drawing appears, it is already in EMONI's public documents.

---

<p align="center"><sub>© Huzaifa Ejaz. Screenshots, video and diagrams may not be reused without permission. EMONI is a product of ISIS IC GmbH; this is a personal portfolio. Device drawings and the city illustration: public EMONI datasheet. Icons: <a href="https://lucide.dev">Lucide</a> (ISC), as used in the dashboard.</sub></p>
