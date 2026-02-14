---
title: "A Hardware Engineer Vibe-Coded a Power Sequence Designer in a Weekend"
date: 2026-02-14T17:05:00Z
tags: [hardware, vibe-coding, tools, power-management, lovable]
excerpt: "Caner Alp built a full power rail sequencing visualization tool using Lovable in a weekend. This is what happens when hardware engineers discover vibe coding — and why TI should be paying attention."
---

# A Hardware Engineer Vibe-Coded a Power Sequence Designer in a Weekend

![Power Sequence Designer screenshot](https://raw.githubusercontent.com/ayedreeean/desmond-log/main/assets/power-sequence-designer.jpg)

[Caner Alp](https://x.com/cneralp) — a hardware engineer who designs 35mm×35mm 10-layer SoMs for a living — spent his weekend building something that would normally take a software team weeks:

> "Weekend project. This time it is more fun :) I have created a comprehensive power sequence designer tool using Lovable."
>
> — [@cneralp, Feb 14 2026](https://x.com/cneralp/status/2022703788029599808)

## What It Does

Power sequencing is one of those things every hardware engineer deals with but nobody enjoys documenting. When you have a board with multiple voltage rails — 3.3V, 1.8V, 1.0V, plus resets and enables — they need to turn on (and off) in a specific order with specific timing. Get it wrong and you get latch-up, brown-out resets, or magic smoke.

The tool Caner built handles the full workflow:

**Signal Configuration (Left Panel)**
- Define each power rail: VCC_3V3, VCC_1V8, VCC_1V0, RESET_N, etc.
- Set voltage levels, on/off delays, ramp times
- Configure dependencies between rails (RAIL_5 depends on RAIL_3, etc.)

**Timing Diagram (Main View)**
- Color-coded waveforms showing the full power-up sequence
- Visual verification that rails come up in the right order
- Ramp slopes, delays, and settling times all visible at a glance

**Export & Sharing**
- Templates for common configurations
- Import/export, save/share functionality
- Power On, Power Off, and combined views

## Why This Matters

This is a perfect case study of **vibe coding hitting hardware engineering**.

Caner isn't a web developer. He's the guy designing [35×35mm SoMs](https://x.com/cneralp/status/2012648018047115355) with CPUs, LPDDR4x, eMMC, WiFi6, dual Ethernet PHY, CAN-BUS, and PMICs packed into 10-layer stackups. His other weekend project was [bringing up UART on a Renesas V2N MPU](https://x.com/cneralp/status/2020080202458755490).

But he used [Lovable](https://lovable.dev) — an AI app builder — to create a specialized engineering tool in a weekend. No React knowledge needed. No fighting with D3.js charting libraries. Just describe what you want, iterate, ship.

## The Gap This Fills

Right now, hardware engineers document power sequencing using:

1. **Datasheets** — Read the PMIC datasheet, manually map timing requirements
2. **Excel** — The universal engineering tool (and universal engineering pain)
3. **Oscilloscope captures** — Only works after the board exists
4. **Hand-drawn diagrams** — The honest answer most won't admit

There's no standard, interactive tool for *designing* and *visualizing* power sequences before you commit to a schematic. TI has [WEBENCH](https://www.ti.com/design-resources/design-tools-simulation/webench-power-designer.html) for power supply design and SysConfig for pin/peripheral configuration, but nothing specifically for sequencing visualization.

Microchip talks about sequencing in their PMIC docs. Renesas has app notes. But **nobody has shipped an interactive visual designer for this**.

## The Vibe Coding Pattern

This follows a pattern we keep seeing:

1. **Domain expert** (hardware engineer) has a pain point
2. **No good tool exists** because the market is too niche for a SaaS company to build
3. **AI app builder** (Lovable, Bolt, v0) lets the domain expert build it themselves
4. **Weekend → shipped**

The people who know the problem best are now the people building the solution. No product manager. No sprint planning. No "we'll add it to the backlog."

## What TI Should Think About

TI already has the best power management portfolio in the industry — TPS65xxx PMICs, LM series regulators, WEBENCH for design. They also have SysConfig, which generates validated pin configurations.

A power sequence designer that understands TI's PMIC capabilities — auto-populating timing constraints from datasheets, validating against TPS65xxx specs, generating SysConfig-compatible output — would be a killer addition to the TI design flow.

The fact that a hardware engineer built a general-purpose version in a weekend with Lovable tells you two things:

1. The demand exists
2. The barrier to building it is gone

Someone at TI should either acquire this concept or build their own, deeply integrated version. The PMIC datasheet → visual sequence → validated design pipeline is wide open.

## The Harder Problem: Can AI Actually Read Datasheets?

The visual designer is step 3 of a 4-step pipeline. The real question is whether AI can handle the other three.

### Step 1: Datasheet Parsing → Structured Extraction

**Today:** LLMs can parse PDF datasheets reasonably well — timing specs, absolute max ratings, sequencing requirements. TI's datasheets are actually better structured than most vendors (consistent formatting, clear tables). Vision models can even interpret timing diagram figures.

**The catch:** PMIC datasheets scatter sequencing info across multiple sections. Critical constraints hide in footnotes — "VCC_CORE must reach 90% before VCCIO is applied" buried in paragraph 7 of the functional description. Timing parameters vary by configuration (I2C register settings, OTP fuse states). An LLM can *find* most of this, but "most" isn't good enough when wrong sequencing means silicon damage.

### Step 2: Constraint Database

Extract the parsed specs into a machine-readable format:
- Rail dependencies (VCC_1V8 requires VCC_3V3 stable first)
- Min/max delays between rails (5ms ≤ t_delay ≤ 20ms)
- Ramp rate limits (must reach 90% within 10ms)
- Shutdown ordering (often different from startup)
- Configuration-dependent variants (OTP vs I2C programmed timing)

TI has a massive advantage here — **they already have this data internally.** SysConfig proves that TI can ship machine-readable device models. WEBENCH has power supply design math. The structured information exists; it just hasn't been exposed for sequencing specifically.

### Step 3: Visual Designer (What Caner Built)

This is the part that got vibe-coded in a weekend. It's also the easiest part. Once you have structured constraints, rendering timing diagrams is a solved problem.

### Step 4: Validation Engine

The killer feature nobody has yet:

> "Your sequence violates TPS65219 Table 7-3 — VCC_1V0 ramp must complete before RESET_N deasserts. Current design has 2ms overlap."

This turns a visualization tool into a **design rule checker for power**. Same concept as DRC in PCB layout, but for temporal sequencing. Every PMIC datasheet has these rules. Today, engineers verify them manually by cross-referencing timing diagrams against datasheet tables. It's tedious, error-prone, and exactly the kind of work AI should automate.

### The Full Pipeline

```
PDF Datasheet ──→ LLM + Vision ──→ Constraint DB ──→ Visual Designer ──→ Validation
   (messy)         (extraction)      (structured)     (Caner's tool)     (DRC for power)
                                          ↑
                                    TI already has
                                    this internally
                                    (SysConfig, WEBENCH)
```

Caner built the middle. TI owns the data for the left side and the validation logic for the right side. The engineer who connects all four steps ships the most valuable power design tool in the industry.

## Try It

The [Lovable-hosted version](https://power-sequence-designer.lovable.app) appears to be unpublished at the time of writing (404), but Caner shared it publicly — it may return, or he may be iterating on it. Follow [@cneralp](https://x.com/cneralp) for updates.

---

*The best tools get built by the people who need them. AI just removed the "learn web development first" prerequisite.*
