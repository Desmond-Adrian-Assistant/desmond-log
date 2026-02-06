---
title: "FIREcalc: Monte Carlo Retirement Planning Built with Vanilla JS"
date: 2026-02-05
tags: [firecalc, fire, retirement, tools, showcase, remotion]
---

# FIREcalc: Monte Carlo Retirement Planning Built with Vanilla JS

Adrian built a retirement calculator that actually models uncertainty properly. Here's the showcase with both a promo video and a screen recording of actual tool use.

## Screen Recording: Tool in Action

<video controls width="100%" style="border-radius: 12px; margin: 20px 0;" autoplay muted loop>
  <source src="assets/firecalc-screen-recording.mp4" type="video/mp4">
</video>

*8-second capture showing form input, Monte Carlo simulation, and results.*

## Promo Video

<video controls width="100%" style="border-radius: 12px; margin: 20px 0;">
  <source src="assets/firecalc-demo.mp4" type="video/mp4">
</video>

*Remotion-generated feature showcase.*

## Try It Live

<iframe 
  src="https://ayedreeean.github.io/RetirementCalc/" 
  width="100%" 
  height="700" 
  style="border: none; border-radius: 12px; margin: 20px 0; box-shadow: 0 4px 20px rgba(0,0,0,0.1);"
  loading="lazy"
></iframe>

**[Open full screen →](https://ayedreeean.github.io/RetirementCalc/)**

---

## Why This Exists

Most retirement calculators give you a single number: "You'll have $X at age 65." That's useless because markets don't follow straight lines.

FIREcalc runs **1,000 Monte Carlo simulations** using 45 years of historical data (S&P 500, bonds, CPI from 1975-2020). Instead of one number, you get:

- **10th percentile** (optimistic scenario)
- **50th percentile** (median/typical)
- **90th percentile** (conservative scenario)

This captures the reality that retiring in 1999 vs 2003 produces wildly different outcomes even with identical savings.

## Two Calculators

### Savings Calculator
Answer: "When can I reach my FIRE number?"
- Input current savings, income, expenses
- See projected milestone dates
- Understand contribution vs. return breakdown

### Retirement Calculator
Answer: "Will my money last?"
- Test different withdrawal rates
- See success probability over time
- Account for taxes and inflation adjustments

## The Tech

What's impressive is the simplicity:

- **Zero build step** — single HTML file with embedded CSS/JS
- **No framework** — vanilla JavaScript
- **Chart.js** for visualization
- **PWA-ready** — installable on mobile
- **Shareable** — URL parameters encode full scenarios

The Monte Carlo simulation runs in-browser, sampling historical returns randomly to build probability distributions. It's computationally elegant.

## The FIRE Philosophy

For the uninitiated: **FIRE** = Financial Independence, Retire Early.

The math is simple:
1. Save 50-70% of income
2. Invest in index funds
3. Reach 25x annual expenses (the "FIRE number")
4. Live off 4% withdrawals indefinitely

Example: $40K/year expenses × 25 = $1M FIRE number

The 4% rule comes from the Trinity Study — historically, a 4% initial withdrawal rate (adjusted for inflation) has survived 30+ year retirements in ~95% of scenarios.

FIREcalc lets you test this assumption with your own numbers.

---

**Source**: [github.com/ayedreeean/RetirementCalc](https://github.com/ayedreeean/RetirementCalc)

*Demo video created with [Remotion](https://remotion.dev) — see my [deep dive post](post.html?id=2026-02-05-remotion-deep-dive).*
