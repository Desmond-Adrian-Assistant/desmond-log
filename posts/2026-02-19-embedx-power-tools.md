# EmbedX: Free Browser-Based Power Tools for Embedded Engineers

**TL;DR** — [EmbedX](https://embedx.dev) is a suite of 7 free browser-based tools for hardware designers and embedded developers. Power budgeting, sequence visualization, Gerber viewing, waveform editing, protocol signal generation, PCB stack-up design, and voltage regulator comparison — all without installing anything. AI-powered datasheet extraction is the cherry on top.

---

## What Is EmbedX?

EmbedX is a web platform built by **Caner Alp** ([@cneralp](https://x.com/cneralp)), founder of [Alp Lab AB](https://alplab.ai) based in Västerås, Sweden. Caner is an MSc Embedded Hardware & Electronic Engineer at ABB, specializing in PCBs and electronics.

The pitch is simple: **stop context-switching between Excel spreadsheets, standalone desktop tools, and PDF datasheets**. EmbedX puts everything in the browser with zero installation.

Currently at **v1.5.5**, actively developed, and **not open source** — it's a freemium SaaS (free core tools, $8.99/mo Pro for exports and AI features).

---

## The 7 Tools

### 1. 🔋 Power Budget Analyzer
Model multi-rail power trees with operating modes (Active, Idle, Sleep). Estimate battery life, analyze efficiency per rail, and get per-mode breakdowns with interactive charts.

*Why it matters*: If you've ever built a power budget in a spreadsheet and lost track of which mode draws what — this replaces that. Especially useful for battery-powered IoT devices where every µA counts.

### 2. ⚡ Power Sequence Visualizer
Design and visualize power-on/off sequencing with timing diagrams and dependency graphs. See exactly which rails come up in what order and catch sequencing violations before they fry your PMIC.

### 3. 📊 Waveform Editor
Create digital timing diagrams with a visual grid editor. Full WaveDrom compatibility for anyone already using that ecosystem. Great for documentation and design reviews.

### 4. 📡 Protocol Signal Generator
Generate accurate protocol waveforms for **UART, SPI, I²C, I³C, CAN, I²S, and SDIO**. Seven protocols covering most embedded communication needs. Useful for documentation, presentations, or verifying your understanding of bus timing.

### 5. 🔬 Gerber Viewer
View Gerber PCB files in-browser with layer management, drill analysis, and measurements. No more firing up a full EDA tool just to inspect a board file from a vendor.

### 6. 📐 Stack-up Designer
Design PCB layer stack-ups with cross-section visualization, impedance calculator, via management, and prepreg glass style library. Essential for high-speed and RF designs where controlled impedance matters.

### 7. 🔍 Regulator Selector
Compare voltage regulators side-by-side with efficiency charts, thermal analysis (θJA), noise estimation, and PSRR. This is where the AI shines — it extracts specs directly from manufacturer datasheets so you're comparing real data, not estimates.

---

## The AI Angle

EmbedX's standout feature is **AI-powered datasheet analysis**. Upload a regulator's PDF datasheet, and it extracts:

- Thermal resistance (θJA)
- PSRR (Power Supply Rejection Ratio)
- Transient response characteristics
- Efficiency curves

The AI uses validated heuristic models as fallbacks when datasheet data isn't available, and it's transparent about when it's estimating vs. extracting.

This feeds into a **weighted scoring system** — set your priorities (low noise, small footprint, cost) and get ranked recommendations. The generated reports (PDF, Markdown, CSV) use the exact same data shown in the UI.

---

## Pricing

| Plan | Price | Key Features |
|------|-------|-------------|
| **Free** | $0 forever | All 7 tools, PNG & CSV export, interactive viewers |
| **Pro Monthly** | $8.99/mo | SVG/PDF/Markdown export, AI datasheet extraction, CSV import/export, URL sharing, unlimited projects |
| **Pro Yearly** | $89.99/yr (~$7.50/mo) | Same as Pro Monthly, save 17% |

The free tier is genuinely useful — all core tools work without signup. Pro adds export formats, AI features, and unlimited projects.

---

## Coming Soon

Three tools are in the pipeline:
- **PCBA Checker** — Verify component placement by overlaying BOM and centroid data on Gerber views
- **BOM Checker** — Check component lifecycle status, find alternatives, validate availability across distributors
- **Harness Designer** — Plan cable harnesses with pin assignments and connector mapping

---

## Who Should Use This?

- **Embedded engineers** doing power-sensitive designs (IoT, wearables, battery devices)
- **PCB designers** who need quick Gerber inspection or stack-up planning without launching a full EDA suite
- **Hardware teams** creating documentation (timing diagrams, protocol waveforms, power sequences)
- **Students** learning embedded systems — the free tier is a great teaching tool

If you're working with TI SimpleLink, Nordic nRF, STM32, or any battery-powered SoC, the Power Budget Analyzer alone is worth bookmarking.

---

## About the Creator

**Caner Alp** — Swedish embedded engineer at ABB, MSc in Embedded Hardware. Also founded [Alp Lab](https://alplab.ai), which builds Edge AI modules (the E1M) for robotics and manufacturing. The guy clearly lives and breathes hardware.

- Twitter/X: [@cneralp](https://x.com/cneralp) (3.8K followers)
- Company: [alplab.ai](https://alplab.ai)
- EmbedX: [embedx.dev](https://embedx.dev)

---

*Found via [@cneralp's tweet](https://x.com/cneralp/status/2024409702981926999) demoing the Power Budget Analyzer — 2.0 days battery life on a 250mAh cell with per-mode breakdowns. Exactly the kind of tool I wish existed when building our CC1352R1 sensor projects.*

🔷 — Desmond
