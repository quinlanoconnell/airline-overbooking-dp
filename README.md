# Airline Overbooking with Dynamic Programming

A full end-to-end implementation of a stochastic dynamic program for airline revenue management. Optimizing overbooking policies, comparing hard-cap vs. flexible pricing strategies, and stress-testing results through sensitivity analysis and Monte Carlo simulation.

---

## Overview

Airlines routinely sell more tickets than they have seats, betting that some passengers will no-show. Set the oversell limit too low and you leave revenue on the table. Set it too high and you're paying denied-boarding penalties.

This project is solved via **backward induction**, covering:

1. Baseline hard-cap overbooking search (oversell allowances of 5–20 coach seats)
2. Comparison against a flexible no-sale policy (4 pricing choices including shutting off sales)
3. Local sensitivity analysis on purchase probabilities
4. Seasonal demand re-solve
5. 100,000-path forward simulation of seasonal policies

---

## Problem Setup

| Parameter | Value |
|---|---|
| Booking horizon | 365 days |
| Coach capacity | 100 seats |
| First-class capacity | 20 seats |
| Coach show-up probability | 95% |
| First-class show-up probability | 97% |
| Upgrade (bump) cost | $50 / passenger |
| Denied-boarding cost | $425 / passenger |
| Annual discount rate | 17% (daily factor β = 1 / (1 + 0.17/365)) |

**Coach pricing actions:** $300 (p = 0.65), $325 (p = 0.45), $350 (p = 0.30)  
**First-class pricing actions:** $425 (p = 0.08), $500 (p = 0.04)

---

## Methodology

### Dynamic Program

The state at each booking day is `(nc, nf)` — the number of coach and first-class tickets already sold. The Bellman equation is:

```
V_t(nc, nf) = max_{a_c, a_f} {
    p_c · price_c + p_f · price_f
    + β · E[V_{t-1}(nc', nf')]
}
```

**Terminal condition:** `V_0(nc, nf) = −E[overbooking cost | nc, nf]`

Terminal costs are computed once via the binomial show-up distribution and stored as a lookup table. Overbooking overflows are resolved by first upgrading bumped passengers to available first-class seats ($50 each), then paying denied-boarding compensation ($425 each) for the remainder.

### Policies Compared

| Policy | Description |
|---|---|
| **Hard-cap** | Fixed oversell ceiling (5–20 extra seats); three pricing choices |
| **Flexible no-sale** | Coach cap raised to 130; airline can also choose to sell *no tickets* on any given day |

### Seasonal Demand

Demand probabilities are scaled by `0.75 + t / 730` where `t` is the booking-day index, creating a ramp from low-demand early in the booking window to high-demand near departure.

### Forward Simulation

The optimal seasonal policies are simulated forward 100,000 times to produce empirical profit distributions, CDFs, overflow probabilities, and denied-boarding rates.

---

## Results Summary

- **Parts 1–2:** Identifies the profit-maximizing oversell cap across the baseline and seasonal demand scenarios.
- **Part 3:** Quantifies whether flexible no-sale control outperforms the best hard-cap rule — and by how much.
- **Part 4:** Stress-tests both policies across ±5 pp coach probability shifts and ±2 pp first-class shifts.
- **Parts 5–6:** Re-solves under seasonal demand and validates DP values via simulation.

All figures and tables are saved to `outputs/` on run.

---

## Repository Structure

```
├── airline_overbooking_project_updated.ipynb   # Main notebook (all parts)
├── outputs/
│   ├── figures/
│   │   ├── policy_comparison_bar.png           # Hard-cap vs. flexible bar chart
│   │   ├── sensitivity_heatmaps.png            # 2-D sensitivity heatmaps
│   │   ├── sensitivity_marginal.png            # Marginal sensitivity curves
│   │   ├── seasonality_profile.png             # Demand multiplier + effective probabilities
│   │   ├── seasonal_profit_hist.png            # Simulated profit density (100k paths)
│   │   └── seasonal_profit_cdf.png             # Empirical CDFs
│   ├── tables/
│   │   ├── baseline_hard_cap_search.csv        # Profit by oversell cap (baseline)
│   │   ├── seasonal_hard_cap_search.csv        # Profit by oversell cap (seasonal)
│   │   ├── policy_comparison.csv               # Hard-cap vs. flexible summary
│   │   ├── sensitivity_hard_cap.csv            # Sensitivity grid (hard-cap)
│   │   └── sensitivity_flexible.csv            # Sensitivity grid (flexible)
│   └── summary.json                            # Compact results payload
└── README.md
```
