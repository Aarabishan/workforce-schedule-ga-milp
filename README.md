# One Day Sewing Trial Workforce Scheduling (GA vs. MILP)

Daily staff scheduling for a single sewing line with 14 sequential operations.
ompare a Genetic Algorithm (GA) against a Mixed-Integer Linear Program (MILP) (PuLP + CBC) under an equal 60-second budget.

Key result (equal 60 s):

- GA: 320.0 finished pieces (target 288 met)

- MILP (CBC): 394.29 finished pieces (time-limited feasible incumbent)

## 🎯 1. Problem Overview

- Goal: Assign 10 operators across 8 working hours to maximize finished pieces at the final operation while respecting eligibility (skills) and process precedence (flow line).
- Operations: 14 stages (e.g., Front panel attach → … → Final QA) with provided SMVs (standard minutes).
- Rates: Operator-operation rates derived as rate = 60 / SMV × speed_multiplier.
- Target: 288 pieces/day (takt ≈ 36 pcs/hour).

Decision variables
- GA chromosome: matrix (𝑁 × 𝐻) = chosen operation index for each operator & hour.
- MILP binaries 𝑥𝑛,ℎ,𝑜: 1 if operator 𝑛 works on operation 𝑜 at hour ℎ; plus linear flow variables 𝑦,𝑌.

## 📦 2. Structure and required libraries

### Cells in order:
   - build E and R, light EDA
   - GA: GA core + convergence plots 
   - MILP: PuLP model (CBC, 60 s cap) 
   - comparison tables/plots
   - reflection

### Minimal requirements.txt:

  - numpy
  - pandas
  - matplotlib
  - pulp==3.3.0
 
## 🧹 3. Data
- operators.csv: operator_id, grade, speed_mult
- operations.csv: op_id, operation_name, smv_minutes
- hours.csv: hour_id (h1..h8)
- target.csv: daily_target_pcs
- eligibility_matrix.csv: E (𝑁×𝑂) in {0,1}
- rate_matrix.csv: R (𝑁×𝑂) from SMV & speed multipliers


## 🛍️ 4. Method

### Genetic Algorithm 

- Representation: (𝑁 × 𝐻) integer grid (op index per operator-hour).
- Fitness: pieces_final − λ·fairness − penalty(target shortfall) with precedence enforced during decode (cumulative throttling).
- Selection: Tournament (k=3–4).
- Crossover: Column-wise (uniform by hour).
- Mutation: Local neighbor shifts respecting eligibility; late-hour bias downstream.
- Extras: Target-aware initialization + late-hour intensification; eligibility repair.

### MILP 

- Modeling: PuLP 3.3.0, CBC solver (open-source).
- Objective: Maximize ∑ 𝑦ℎ,𝑂(final throughput).
- Constraints: assignment (≤1 per operator-hour), eligibility, linear linking 𝑦=𝑅⋅𝑥, cumulative flow, precedence 𝑌ℎ,𝑜 ≤ 𝑌ℎ,𝑜−1 , and hard demand (≥ 288).
- Practical settings: 60 s time limit, optional warm-start from GA schedule.

> 🔴 Note:  the GA model was re-run by setting to 60s in order to check the computational time, solution quality and equal-time comparison with MILP


## 📊 5. Results (equal 60-second budget)

| Method              | Pieces @ Final | Demand Met | Status                       | Time (s) |
| ------------------- | -------------: | :--------: | ---------------------------- | -------: |
| **GA (60s cap)**    |     **320.00** |      ✅     | Feasible (anytime heuristic) | **60.1** |
| **MILP (CBC, 60s)** |     **394.29** |      ✅     | Not Solved (stopped on time) | **60.0** |


Hourly profile (MILP): back-loaded finishing with spikes at hours ~6–7, reflecting WIP build-up then strong staffing at bottleneck stages (e.g., Final topstitch/QA).

### Figures
- GA convergence (best/mean fitness vs generations)
- GA schedule heatmap (operators × hours)
- MIP schedule heatmap (operators × hours)
- Hourly finished pieces at final (GA & MIP)


## 💡6. Discussion/Highlights

- Quality: Under equal time, MILP produced a higher-quality feasible schedule (394.29 > 320).
- Time & scalability: GA is anytime and scales predictably with pop×gens; MILP can find strong incumbents quickly but may hit time limits as size/tightness grows.
- Limitations: synthetic data, single-day horizon, no uncertainty or setup costs; GA precedence via decode; MILP fairness omitted (empirically inactive).
- Improvements: hybrid GA→MILP (warm-starts), parallel GA fitness, metaheuristic post-improvement (SA/Tabu/LNS), takt/WIP smoothing penalties, robust/stochastic variants.


## 📦 7. Reproducibility

- Fixed random seeds used in GA examples.
- All generated artifacts (*_best_schedule.csv, *_kpis.txt, plots) are saved under outputs/.


## 🧾 8.Citation / Acknowledgements

- Solver: PuLP 3.3.0 with CBC.
- Python: NumPy, Pandas, Matplotlib.
- This repository is for MSc coursework; data are illustrative.

🎓 Acadamice context: This repository documents one of the applied projects completed for the Machine Learning module in my MSc in Data Science programme (Octorber 2025). All results, code, and design choices are provided for educational purposes; actual performance may vary across environments and dataset revisions.
