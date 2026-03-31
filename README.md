# Dynamic SIP Simulator

A single-file financial simulator that compares Fixed vs Dynamic SIP (Systematic Investment Plan) strategies — built to answer a question most retail investors never properly model: **does timing your monthly investment around market dips actually beat a flat SIP, on an equal-capital basis?**

Live demo → [link]

---

## What it does

Most SIP comparisons are unfair. Dynamic strategies invest more during dips, so naturally they deploy more capital — and more capital means higher absolute returns regardless of strategy quality. This simulator enforces **strict equal-capital accounting**: whatever Dynamic doesn't put into equity is parked in a liquid fund at your chosen rate. Both strategies always deploy the same total rupee amount each month. The winner genuinely earned it.

**Two modes:**

**Backtest** — runs your strategy against real Nifty 50 monthly closes from November 1994 to March 2026 (377 months). Every valid rolling window for your chosen horizon is tested. You get win rates, median outperformance, and a distribution of outcomes — not a single cherry-picked period.

**Simulate** — generates synthetic market paths using Itô-corrected GBM, letting you stress-test across Bear / Sideways / Bull / Chaotic regimes with up to 100,000 Monte Carlo runs. Useful for understanding how volatility and trigger sensitivity interact before committing to a backtest conclusion.

---

## How the Dynamic strategy works

Each month, the simulator compares the current NAV to a trailing moving average (configurable: 1–12 months). 

- If NAV is more than X% below the average → **dip detected** → invest up to your configured ceiling (default: 2× base SIP)
- If NAV is more than Y% above the average → **rise detected** → invest down to your configured floor (default: 0.5× base SIP)
- Otherwise → invest the base SIP amount

The multiplier scales with magnitude: each additional full multiple of the trigger threshold adds a configurable step (default 10%) to the investment multiplier. Larger crashes → proportionally larger response, capped at your ceiling.

The trailing average uses **past months only** — no look-ahead bias.

---

## Returns methodology

XIRR is computed via Newton-Raphson with multiple starting guesses (positive, negative, and wide) plus a bisection fallback. Each monthly investment is an outflow dated to that month; the terminal portfolio value is the inflow. This is the same methodology used by AMFI and most Indian fund platforms — it correctly handles irregular cashflows, step-up SIPs, and loss scenarios where a positive-only initial guess diverges.

---

## Key parameters

| Parameter | What it controls |
|-----------|-----------------|
| Base monthly SIP | The fixed capital commitment. Both strategies always deploy this total. |
| Dip ceiling | Max equity investment in a dip month (₹ absolute, not a multiplier) |
| Rise floor | Min equity investment in a rally month (₹ absolute) |
| Dip / Rise trigger | How far from the trailing average before the signal fires (5–25%) |
| Avg window | Months of history used to compute the baseline (1–12) |
| Annual step-up | SIP grows by this % each year — applied equally to both strategies |
| Liquid rate | Annual return on parked capital (0% = worst case, 4% ≈ liquid fund, 7% ≈ debt fund) |
| Horizon | 3–30 years |

---

## What I built this to learn

I built this as a side project to pressure-test an intuition I'd formed from reading about value averaging and momentum-adjusted SIPs. The specific question: **what's the actual edge of dynamic allocation, net of the liquidity cost of parking capital?** The answer depends heavily on volatility regime, trigger sensitivity, and horizon — and most popular articles collapse those dimensions into a single backtest number.

Building the simulator forced me to think carefully about:
- **Fair comparison design** — the equal-capital constraint took three iterations to get right
- **XIRR correctness** — the standard Newton-Raphson implementation diverges in bear markets; multi-guess + bisection fallback was necessary
- **Backtest honesty** — showing the full distribution of rolling windows, not just the median or a recent period
- **UI for non-experts** — rupee-anchored controls (not abstract multipliers), inline tooltips explaining each parameter in plain language, a "How it works" panel that doesn't assume financial literacy

---

## Stack

Single HTML file. Vanilla JS. No frameworks, no build step, no dependencies — just `<canvas>` for charts and a bit of math.

The Nifty 50 data (Nov 1994–Mar 2026, 377 monthly closes) is embedded directly. CSV upload is supported for other indices.

---

## What the results show (spoiler)

Across 10-year Nifty rolling windows with default parameters, Dynamic SIP beats Fixed SIP in roughly **60–65% of periods** — but the median outperformance is modest (~0.3–0.6 XIRR percentage points). The tail behaviour is more interesting: Dynamic loses badly in strongly trending bull markets (it consistently under-invests during the run-up) and wins meaningfully in high-volatility, mean-reverting regimes. The liquid fund rate matters more than most people expect.

The simulator lets you find where your own conviction should sit.

---

## Running it

Download `sip-simulator.html`. Open in any browser. No server required.
