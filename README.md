# MLB Schedule Optimization — DASC 2026

This repository contains the work for our **Mixed-Integer Linear Programming (MILP)** project to optimize the Major League Baseball (MLB) schedule. The primary goal is to minimize total league-wide travel distance across the 162-game season, subject to real-world scheduling constraints such as mandatory series lengths, matchup quotas, and consecutive home/away limits.

---

## Please Read

I (Branson Vu) may extend the EDA to incorporate uncertainty (e.g., weather conditions, unpredictable real-world events), in which case I may add a Bayesian component — modeling attendance, rivalry strength, and posterior-weighted objective coefficients using R (RStan). This is not yet implemented.

---

## Repository Structure

```
OptimizationMethods2026/
│
├── Optimization_EDA.ipynb          # Full-season EDA — parameter extraction (start here)
├── OP.ipynb                        # Pyomo MILP model — AL East prototype (Google Colab)
│
├── al_east_schedule.csv            # AL East-only filtered schedule (model input)
├── al_east_distance_matrix.csv     # AL East 5x5 stadium distance matrix (model input)
│
└── alldata_2025/                   # Raw Retrosheet 2025 season data
    ├── gamelogs/gl2025.txt         # Full game-by-game log (2,430 games)
    ├── schedules/2025schedule.csv  # Full season schedule
    └── teams/TEAM2025              # Team ID reference
```

---

## File Descriptions

### `Optimization_EDA.ipynb` — Exploratory Data Analysis
The **starting point** for anyone joining the project. Processes 2025 Retrosheet data to extract and validate every parameter needed to build the MILP model. Covers four tasks:

| Task | Output | MILP Role |
|------|--------|-----------|
| **Task 1** — Sets & Mappings | 30 teams (`T`), 30 stadiums (`C`), home-park map (`H_a`) | Index sets and home-location constraints |
| **Task 2** — Matchup Matrix | 30×30 integer matrix `M(a,j)` | Fixes game counts: `Σ G(a,j,d) = M(a,j)` |
| **Task 3** — Distance Matrix | 30×30 Haversine distance matrix `Dist(b,c)` | Objective function coefficients |
| **Task 4** — Baseline Travel | League-wide historical miles traveled | Upper bound / benchmark for the optimizer |

---

### `OP.ipynb` — Pyomo MILP Model (AL East Prototype)
The **optimization model**, built and run in **Google Colab** using Pyomo with the CBC solver. Scoped to the AL East division (5 teams: BAL, BOS, NYA, TBA, TOR) as a proof-of-concept before scaling to all 30 teams.

**What it does:**
- Loads `al_east_schedule.csv` and `al_east_distance_matrix.csv`
- Defines Pyomo sets (`T`, `C`, `D`), parameters (`Dist`, `H`, `M`), and binary decision variables (`A`, `G`, `L`)
- Minimizes `Σ Dist(b,c) · A(a,b,c,d)` subject to location flow, matchup, and single-game constraints
- Outputs an optimized schedule and compares it against the real 2025 baseline

**Key result (AL East):**
| Metric | Value |
|--------|-------|
| Actual 2025 travel | 35,774.9 miles |
| Optimized travel | 12,745.2 miles |
| Savings | **23,029.8 miles (64.4%)** |

> **Note:** Run this notebook in [Google Colab](https://colab.research.google.com/) — it uses `google.colab.files` for uploading the two CSV inputs.

---

### `al_east_schedule.csv` — AL East Schedule (Model Input)
Filtered subset of the 2025 full-season schedule containing only AL East division matchups (BAL, BOS, NYA, TBA, TOR). 131 games. This is the schedule the MILP model optimizes against.

### `al_east_distance_matrix.csv` — AL East Distance Matrix (Model Input)
A 5×5 symmetric matrix of Haversine distances (miles) between the five AL East home stadiums: `TOR02`, `BAL12`, `BOS07`, `TAM02`, `NYC21`.

---

## Project Status
- [x] EDA completed — all 30-team MILP parameters extracted and validated
- [x] MILP model built — AL East prototype implemented in Pyomo (CBC solver)
- [x] AL East results computed — 64.4% travel reduction vs. 2025 baseline
- [ ] Scale model to full 30-team league
- [ ] Solver integration with Gurobi (for speed at full scale)
- [ ] Results analysis and comparison vs. full-season baseline

---

## Quick Reference — Key Numbers
- **Teams (full model):** 30
- **Teams (prototype):** 5 (AL East)
- **Total games (full season):** 2,430
- **AL East games modeled:** 131
- **Baseline travel (AL East):** 35,774.9 miles
- **Optimized travel (AL East):** 12,745.2 miles

---

## Getting Started

**For EDA:** Open `Optimization_EDA.ipynb` locally in Jupyter. Requires `pandas`, `numpy`, `matplotlib`, `seaborn`.

**For the MILP model:** Open `OP.ipynb` in [Google Colab](https://colab.research.google.com/), then upload `al_east_schedule.csv` and `al_east_distance_matrix.csv` when prompted.
