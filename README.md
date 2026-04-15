# MLB Schedule Optimization — DASC 2026

This repository contains the work for **Mixed-Integer Linear Programming (MILP)** project to optimize the Major League Baseball (MLB) schedule. The primary goal is to minimize total league-wide travel distance across the 162-game season, subject to real-world scheduling constraints such as mandatory series lengths, matchup quotas, and consecutive home/away limits.

## Please Read 

I (branson vu), might add more to the EDA should we focus on designing a schedule that involves uncertainty (i.e weather conditions or unpredictable real world events), which then I may want to implement Bayesian Methods based on attendance, rivaly strength and using posterior weights on the objective model using R (rstan package).

---

## What's in This Repo

### `Optimization_EDA.ipynb` — Exploratory Data Analysis
This notebook is the **starting point for any of you working on the project**. It processes 2025 Retrosheet game log data to extract and validate all parameters needed to build the MILP model. It covers four tasks:

| Task | Output | MILP Role |
|------|--------|-----------|
| **Task 1** — Sets & Mappings | 30 teams (`T`), 30 stadiums (`C`), home-park map (`H_a`) | Index sets and home-location constraints |
| **Task 2** — Matchup Matrix | 30×30 integer matrix `M(a,j)` | Fixes game counts: `Σ G(a,j,d) = M(a,j)` |
| **Task 3** — Distance Matrix | 30×30 Haversine distance matrix `Dist(b,c)` | Objective function coefficients |
| **Task 4** — Baseline Travel | League-wide historical miles traveled | Upper bound / benchmark for the optimizer |

### `alldata_2025/` — Raw Data
Contains Retrosheet game logs, schedules, rosters, and team files for the 2025 MLB season. The key files used in the EDA are:
- `gamelogs/gl2025.txt` — Full game-by-game log
- `teams/TEAM2025` — Team ID reference
- `schedules/2025schedule.csv` — Full season schedule

---

## Project Status
- [x] EDA completed — all MILP parameters extracted and validated
- [ ] MILP model construction (in progress)
- [ ] Solver integration (Pyomo)
- [ ] Results analysis and comparison vs. baseline

---

## Quick Reference — Key Numbers
- **Teams:** 30
- **Stadiums:** 30 home parks
- **Total games:** 2,430
- **Baseline travel:** computed in Task 4 of the EDA notebook (league-wide miles, 2025 season)

---

## Getting Started
All EDA code runs with standard Python libraries (`pandas`, `numpy`, `matplotlib`, `seaborn`). Open `Optimization_EDA.ipynb` in Jupyter and run all cells top-to-bottom.
