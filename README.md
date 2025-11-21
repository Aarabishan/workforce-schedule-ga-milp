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

