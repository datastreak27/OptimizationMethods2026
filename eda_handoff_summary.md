# MLB Schedule Optimization — EDA Summary & Model Handoff

## What This Notebook Does

The notebook performs four structured EDA tasks on the **2025 MLB Retrosheet game log data** to extract, validate, and visualize every parameter needed for the MILP schedule optimizer. 

---

## The Four Tasks at a Glance

### Task 1: Define the Sets & Home Stadium Mapping

**What it produces:** Two index sets and one mapping dictionary.

| Output | Description | Size |
|--------|-------------|------|
| **$T$** | All 30 MLB team codes (e.g., `SEA`, `NYA`, `CHN`) | 30 |
| **$C$** | All active stadium IDs (e.g., `SEA03`, `NYC21`) | 30 home + neutral sites |
| **$H_a$** | Dictionary mapping each team → its primary home park | 30 key-value pairs |

**Key takeaway for the modeler:**
- $H_a$ is derived by finding the park where each team hosts the *most* games, which correctly filters out neutral-site games (e.g., the Tokyo series at `TOK01`).
- This dictionary is plugged directly into the constraint that "pins" a team's location to their home park on home-game days.
- The Sacramento Athletics (`ATH → SAC01`) is a 2025-specific mapping — the A's relocated from Oakland.

---

### Task 2: The Matchup Matrix ($M_{a,j}$)

**What it produces:** A **30×30 integer matrix** where row $a$ (away team) and column $j$ (home team) gives the number of games team $a$ must play *at* team $j$'s home.

**Key takeaways for the modeler:**

- **The diagonal is all zeros** — a team never visits itself. This is a built-in sanity check.
- **Division opponents have higher counts (~6–7 games)** vs. inter-league matchups (~3–4). This asymmetry means the optimizer can't just uniformly redistribute games.
- **The matrix sums to exactly 2,430** (= 30 teams × 81 home games each). This is the total number of games in the season and serves as a checksum.
- **Each row sums to ~81** (away games) and **each column sums to ~81** (home games). If any team deviates, it flags data quality issues.
- This matrix is passed *verbatim* into the constraint: $\sum_{d \in D} G_{a,j,d} = M_{a,j}$. It is **not** a variable — it is a **fixed parameter**.

> [!IMPORTANT]
> The matchup matrix is the single most important constraint input. It guarantees the optimized schedule has the exact same matchup structure as the real season — the optimizer only rearranges *when* and *in what order* games happen, not *which* games happen.

---

### Task 3: The Distance Matrix ($Dist_{b,c}$)

**What it produces:** A **30×30 symmetric matrix** of pairwise Haversine (great-circle) distances in miles between all home stadiums.

**Key takeaways for the modeler:**

- **This is the objective function's coefficient.** Every unit of travel has a cost = $Dist_{b,c}$, and the optimizer minimizes the sum of all these costs across all teams and days.
- **Distances range from ~8 miles** (Yankee Stadium ↔ Citi Field, both in NYC) **to ~2,400+ miles** (Seattle ↔ Miami). This enormous range means the optimizer has significant room to reduce travel by clustering geographically close series together.
- **The mean pairwise distance is ~1,300–1,400 miles**, and the distribution is roughly uniform / slightly right-skewed — there is no single dominant cluster.
- **Haversine underestimates actual road/flight distance by ~10–15%**, but it preserves the relative ranking of distances, which is what matters for optimization. If the colleague wants more precision, they can swap in geodesic or actual flight-mile data without changing the model structure.
- **Tokyo (`TOK01`) is excluded** from the distance matrix since the MILP models domestic travel only. International games should be handled as fixed schedule blocks.

---

### Task 4: The Baseline Objective Value

**What it produces:** The **actual total miles traveled** by all 30 teams in the 2025 season, reconstructed from the chronological game log.

**Key takeaways for the modeler:**

- **This number is the benchmark.** If the MILP solver outputs a schedule with total travel *less* than this baseline, the optimization succeeded. The improvement percentage is: $\frac{\text{Baseline} - \text{Optimized}}{\text{Baseline}} \times 100\%$
- **West Coast teams (SEA, ATH, ANA) travel the most** because they are geographically isolated — they must fly cross-country for every Eastern road trip. The optimizer likely can't dramatically reduce their travel, but it can smooth out unnecessary back-and-forth.
- **Central/Eastern teams travel less** because the stadium density is higher east of the Mississippi.
- **The travel calculation includes the "return home" leg** at season's end but **excludes international neutral-site games** (Tokyo). The modeler should adopt the same convention.
- The per-team breakdown also reveals **which teams have the most "inefficient" historical schedules** (e.g., coast-to-coast-to-coast zigzags). These are the teams where the optimizer should yield the biggest gains.

> [!TIP]
> The league-wide baseline total is the **upper bound** for the MILP objective. Set it as a known feasible solution (warm start) if your solver supports it — this can dramatically speed up branch-and-bound.

---

## What the Modeler Needs from This EDA

Here's the concrete deliverable list:

| Deliverable | Format | Variable Name |
|---|---|---|
| Team set | Python list of 30 strings | `T` |
| Home-park mapping | `dict[str, str]` — team → park ID | `H_a` |
| Matchup matrix | `pd.DataFrame` (30×30 int) | `M_aj` |
| Distance matrix | `pd.DataFrame` (30×30 float, miles) | `dist_by_team` |
| Baseline total travel | Single integer (miles) | `league_total` |
| Per-team travel breakdown | `pd.DataFrame` with team + miles | `travel_df` |

All of these are computed and ready to export as `.csv` or `.pkl` from the notebook.

---

## Assumptions & Caveats to Communicate

1. **Distances are Haversine (as-the-crow-flies)**, not actual flight paths. Fine for relative optimization; not for quoting exact mileage savings.
2. **The game log may be a partial season** if the data was pulled mid-season. Check that the total game count matches expectations (2,430 for a full season).
3. **Doubleheaders appear as separate rows** in the game log. They don't generate extra travel (same park both games), and the travel function handles this correctly.
4. **Postponed/rescheduled games** are included in the log at their *played* date, which is the right behavior for computing actual historical travel.
5. **The model assumes 3-game series** as the standard unit. The EDA doesn't enforce this — that's a constraint the modeler adds.

## Please Read 

I (branson vu), might add more to the EDA should we focus on designing a schedule that involves uncertainty (i.e weather conditions or unpredictable real world events), which then I may want to implement Bayesian Methods based on attendance, rivaly strength and using posterior weights on the objective model using R (rstan package).