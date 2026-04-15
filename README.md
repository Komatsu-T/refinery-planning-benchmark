# refinery-planning-benchmark
This repository provides a Python implementation of a production planning benchmark based on real-world refinery-petrochemical complex data. The original repository is
https://github.com/EMRPS/refinery-planning-benchmark. In the original repository, computational results were obtained from global optimization solvers ANTIGONE and BARON. However, since ANTIGONE and BARON require paid licenses, this repository solves the problem using free solvers. This repository contains three types of problems as described below.

## Repository structure
```
.
├── README.md # Repository overview and structure
├── case1 # parameters, and computational results for case 1
├── case2 # For case 2
└── case3 # For case 3
```

## Case 1
### Problem setting summary
A refinery purchases 31 types of crude oil and produces 32 types of products (gasoline, diesel, jet fuel, etc.) through distillation, secondary processing, and blending. The objective is to maximize profit (product revenue − crude oil cost) by determining how much of each crude to buy, how to operate 132 processing units, and how to blend final products.

There are 3,573 decision variables (flow rates, quality values, and operating modes for each stream — all continuous), 3,428 constraints (mass balance, quality tracking, capacity limits, product specifications, etc.), and a single planning period (steady-state snapshot).

The problem type is NLP (Nonlinear Programming). There are no integer variables, so it is neither MIP nor MINLP. The equation "flow rate × quality value = quality mass flow" introduces bilinear terms (variable × variable) in 289 constraints. This makes the feasible region non-convex.

### Files in the `case1` folder

The following files are from the original repository for solving problems with ANTIGONE and BARON:
- all_parameters.gdx
- all_parameters.txt
- all_sets.gdx
- all_sets.txt
- case1.gms
- log_antigone.log
- log_baron.log
- solution_antigone.gdx
- solution_baron.gdx

The following files contain detailed problem information:
- case1_structure.md (written in Japanese)
- case1_structure_en.md (written in English)
