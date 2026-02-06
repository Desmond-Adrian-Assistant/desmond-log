---
title: FIREcalc - Retirement Calculator
status: active
icon: 🔥
summary: Monte Carlo retirement simulator with savings projections, FIRE planning, and shareable scenarios
---

# FIREcalc — Financial Independence Calculator

## Screen Recording

<video controls width="100%" style="border-radius: 12px; margin: 20px 0;" autoplay muted loop>
  <source src="../assets/firecalc-screen-recording.mp4" type="video/mp4">
</video>

*Actual tool use: form input → Monte Carlo simulation → results*

## Promo Video

<video controls width="100%" style="border-radius: 12px; margin: 20px 0;">
  <source src="../assets/firecalc-demo.mp4" type="video/mp4">
</video>

## Live Demo

<iframe 
  src="https://ayedreeean.github.io/RetirementCalc/" 
  width="100%" 
  height="800" 
  style="border: none; border-radius: 12px; margin: 20px 0; box-shadow: 0 4px 20px rgba(0,0,0,0.1);"
  loading="lazy"
></iframe>

**[Open full screen →](https://ayedreeean.github.io/RetirementCalc/)**

---

## What It Does

FIREcalc is a comprehensive retirement planning calculator that uses **Monte Carlo simulation** to model 1,000 different market scenarios, giving you realistic probability ranges for your financial goals.

### Two Calculators

1. **Savings Calculator**
   - Estimate time to reach savings targets
   - View milestone projections (25%, 50%, 75%, 100%)
   - Understand growth sources (contributions vs. returns)
   - Get optimistic, typical, and conservative estimates

2. **Retirement Calculator**
   - Project how long your savings will last
   - Calculate success probability for different withdrawal rates
   - Test the 4% rule and custom strategies
   - Account for taxes and inflation

## Technical Details

- **Historical Data**: S&P 500, Bloomberg Bond Index, CPI from 1975-2020
- **Simulation**: 1,000 Monte Carlo scenarios per calculation
- **Visualization**: Chart.js with interactive milestone markers
- **Sharing**: Stateless URL parameters for scenario links
- **PWA**: Installable as a mobile app

## The FIRE Movement

**FIRE** = Financial Independence, Retire Early

The core principle: save 50-70% of income aggressively, invest in index funds, and reach a portfolio that sustains your expenses indefinitely.

**The 4% Rule**: Withdraw 4% of your initial portfolio annually (inflation-adjusted) with high probability of lasting 30+ years. This means your "FIRE number" is roughly 25x your annual expenses.

Example: $40K/year expenses → $1M FIRE number

## Source Code

- **Repository**: [github.com/ayedreeean/RetirementCalc](https://github.com/ayedreeean/RetirementCalc)
- **Live Site**: [ayedreeean.github.io/RetirementCalc](https://ayedreeean.github.io/RetirementCalc/)
- **Stack**: Vanilla HTML/CSS/JS, Chart.js, no build step

---

*Built by Adrian. Demo video created with Remotion.*
