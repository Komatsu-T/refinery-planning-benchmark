# Case 1 — Overall Problem Structure

## Overview

**File**: `case1.gms` (314 KB, 7,082 lines)
**Problem type**: NLP (Nonlinear Programming) — no integer variables
**Objective**: Maximize profit (= product revenue − crude oil purchase cost)
**Solver directive**: `Solve m using MINLP maximizing x3573;`

---

## Problem Scale

| Item | Value |
|------|-------|
| Decision variables | 3,573 (all continuous) |
| Constraints | 3,428 |
| Non-zero elements | 12,038 (of which 2,082 are nonlinear) |
| Fixed variables (fx) | 359 |

---

## Role of Related Files

| File | Contents |
|------|----------|
| `case1.gms` | Scalar-form optimization model. Variables are x1–x3573; constraints are e1–e3428. Can be run directly in GAMS |
| `all_sets.txt` | Semantic set definitions: streams (S), units (U), batches (M), quality attributes (Q), etc. — 25 types in total |
| `all_parameters.txt` | Numerical parameters: prices, yields, quality bounds, capacity constraints, etc. — 28 types in total |
| `all_sets.gdx` / `all_parameters.gdx` | GAMS binary format (GDX) of the above |
| `log_baron.log` | BARON solver log (5 hours; best solution ≈ 34.17 million) |
| `log_antigone.log` | ANTIGONE solver log |
| `solution_baron.gdx` / `solution_antigone.gdx` | Solutions from each solver (GDX binary) |

**Note**: `case1.gms` is in "scalar form" produced by GAMS Convert, so variable names are simply numbered (x1, x2, …). To recover the original stream names (s0, s1, …) and unit names (UCDU0, Upf3, …), one must cross-reference `all_sets.txt` and `all_parameters.txt`.

---

## Decision Variable Structure (x1 – x3573)

Analysis of the variable indices in `case1.gms` yields the following classification.

### Variable Index Mapping

| Index Range | Count | Meaning | Description |
|---|---|---|---|
| x0 – x364 | 364 | **Stream flow rates** | Mass flow rate of each stream (tons/period). e.g., x11 ≈ flow of stream s10, x271 ≈ flow of stream s270. These appear directly in the objective function with product-price / feedstock-cost coefficients |
| x365 – x700 | 336 | **CDU & processing-unit auxiliary variables** | CDU cut-point controls, batch selection ratios, splitter split fractions, etc. — variables governing unit operations |
| x701 – x1245 | 545 | **Quality mass flow rates** | Results (z) of bilinear equations `z = a × b`. Computed as "flow rate × quality value" (e.g., sulfur mass flow = total flow × sulfur concentration) |
| x1246 – x2300 | 1,055 | **Stream quality values** | Tracked quality attributes for each stream: specific gravity (Q0), octane number (Q2), sulfur content (Q11), etc. These appear as factor b in bilinear terms |
| x2301 – x3473 | 1,173 | **Yield / mode / other auxiliary variables** | Yield deviations in the delta-base formulation, batch weights, capacity-constraint auxiliary variables, etc. |
| x3474 – x3572 | 99 | **Quality ratio variables** | Factor a in bilinear terms. Ratio variables used to compute quality values at mixer outlets |
| x3573 | 1 | **Objective function (profit)** | The quantity to be maximized |

### Objective Function (e3428) Structure

```
e3428: -Σ(product price × product flow) + Σ(feedstock cost × feedstock flow) + x3573 = 0
```

Rearranging: `x3573 = Σ(product price × product flow) − Σ(feedstock cost × feedstock flow)`, so `maximize x3573` = maximize profit.

#### Revenue Terms (22 terms) — product stream flow × unit price

| Variable | Corresponding Stream | Product Price | Notes |
|----------|---------------------|--------------|-------|
| x271 | s270 | 7,981,000 | Highest-value product |
| x149 | s148 | 7,535,000 | |
| x16 | s15 | 7,488,000 | |
| x323 | s322 | 7,328,000 | |
| x324 | s323 | 7,167,000 | |
| x206 | s205 | 6,974,000 | |
| x58 | s57 | 6,899,000 | |
| x322 | s321 | 6,809,000 | |
| x321 | s320 | 6,218,000 | |
| x54 | s53 | 5,527,000 | |
| x59 | s58 | 5,433,000 | |
| x68 | s67 | 5,217,000 | |
| x201 | s200 | 4,992,000 | |
| x67 | s66 | 4,897,000 | |
| x61 | s60 | 4,850,000 | |
| x177 | s176 | 4,715,000 | |
| x220 | s219 | 4,710,000 | |
| x11 | s10 | 4,400,000 | |
| x169 | s168 | 4,322,000 | |
| x198 | s197 | 2,932,000 | |
| x190 | s189 | 2,558,000 | |
| x272 | s271 | 846,000 | Lowest-value product |

※ The remaining 15 products have price = 1 and are treated as by-products / waste.

#### Cost Terms (31 terms) — feedstock stream flow × unit cost

| Variable | Corresponding Stream | Feedstock Cost | Notes |
|----------|---------------------|---------------|-------|
| x376 | s11 | 14,468,000 | Most expensive feedstock (specialty crude) |
| x378 | s13 | 6,646,000 | |
| x386 | s21 | 5,487,000 | |
| x372 etc. | s7, s42, s63, s77, etc. | 5,547,000 | Standard crudes (25 types, same price) |
| x385 | s20 | 4,691,000 | |
| x377 | s12 | 4,245,000 | Least expensive feedstock |
| x384 | s19 | 4,850,000 | |

---

## Constraint Structure (e1 – e3428)

### Constraint Classification

| Type | Count | Symbol | Description |
|------|-------|--------|-------------|
| Bilinear equalities | 289 | =E= | Quality tracking (flow × quality = quality mass flow) |
| Linear equalities | 2,163 | =E= | Mass balance, yield calculations, quality transformations |
| Upper-bound inequalities | 908 | =L= | Quality upper bounds, capacity limits, flow upper bounds |
| Lower-bound inequalities | 68 | =G= | Quality lower bounds, flow lower bounds, minimum production |
| **Total** | **3,428** | | |

### Constraint Details

#### 1. Bilinear Equalities (around e1–e289) — Quality Tracking

```
e1:  x3481 * x1293 - x1095 = 0
```

Meaning: `quality ratio (x3481) × quality value (x1293) = quality mass flow (x1095)`

When streams A (flow fA, sulfur concentration qA) and B (flow fB, sulfur concentration qB) are blended in a mixer, the outlet sulfur concentration is `(fA×qA + fB×qB) / (fA+fB)`. The numerator term `fA×qA` is a bilinear term and is **the source of non-convexity in this problem**.

#### 2. Linear Equalities — Mass Balance (around e534–)

```
e534:  x1 - x366 - x367 = 0      (input = sum of outputs)
e538:  x239 - x604 - ... - x609 = 0  (splitter distribution)
```

Meaning: At each unit node, "total inflow = total outflow." This is conservation of mass.

#### 3. Linear Equalities — Yield Relationships (around e290–)

```
e290:  x12 - x1211 = 0
```

Relates the input flow and output flow of a processing unit. For fixed-yield units (Upf), output = yield coefficient × input.

#### 4. Inequalities — Flow & Quality Bounds (around e3380–e3428)

```
e3380:  x14 >= 2         (minimum flow for a stream)
e3386:  x3  <= 0.061     (maximum flow for a stream)
e3388:  x11 <= 0.8       (maximum sales volume for a product)
```

---

## Refinery Network Structure

### Unit Inventory (132 units)

| Unit Type | Count | Example Names | Role |
|-----------|-------|---------------|------|
| CDU (Crude Distillation Unit) | 2 | UCDU0, UCDU1 | Separates crude oil into fractions by boiling point |
| Fixed-yield processing units (Upf) | 20 | Upf0–Upf19 | Reforming, hydrotreating, etc. Output is a fixed ratio of input |
| Delta-base processing units (Upd) | 5 | Upd0–Upd4 | FCC, etc. Yield varies nonlinearly with input quality |
| Mixers (Umix) | 32 | Umix0–Umix31 | Merge multiple streams |
| Splitters (Usplit) | 61 | Usplit0–Usplit60 | Split one stream into several |
| Blenders (Ublend) | 12 | Ublend0–Ublend11 | Final product blending (subject to quality specifications) |

### Stream Classification (364 streams)

| Category | Count | Description |
|----------|-------|-------------|
| Feedstock streams (S_M) | 31 | Crude oils entering from outside (s7, s11, s12, s13, etc.) |
| Product streams (S_P) | 37 | Final products leaving the refinery (s10, s15, s53, etc.) |
| Intermediate streams | ~296 | Internal flows connecting units |

### CDU Input/Output Example

**CDU0 (UCDU0)**:
- Inputs: 25 streams (supply lines from individual crude tanks)
- Outputs: 10 streams (light ends, naphtha, kerosene, gas oil, residue, etc.)
- Yield parameter: y(UCDU0, m0, s122, s8) = 0.0319 (yield of fraction s122 from crude s8 ≈ 3.2%)

---

## Parameter List (all_parameters.txt)

| Parameter | Description | Number of Entries |
|-----------|-------------|-------------------|
| c_P(s) | Product price (currency/ton) | 37 |
| c_M(s) | Feedstock cost (currency/ton) | 31 |
| FVMin(s,t) | Minimum flow rate for stream s | 8 |
| FVMax(s,t) | Maximum flow rate for stream s | 42 |
| FQ0(s,q) | Fixed quality value for stream s | 546 |
| FQMin(s,q) | Quality lower bound | 10 |
| FQMax(s,q) | Quality upper bound | 38 |
| FQcut(m,s,ss,q) | CDU cut-point–quality relationship | 5,550 |
| y(u,m,s,ss) | Yield coefficient (fraction from input ss to output s in unit u, batch m) | 625 |
| phi(u,m,s,ss) | Swing-cut ratio | 19 |
| gamma(u,m,s) | Delta-base reference yield | 297 |
| delta(u,m,s,q) | Delta-base yield deviation | 53 |
| B(u,m,q) | Delta-base reference quality value | 15 |
| Del(u,m,q) | Delta-base quality deviation step size | 15 |
| alpha(s,ss,q) | Quality transfer ratio (quality transfer between streams) | 70 |
| w(u,m,s,q) | Component composition | 7 |
| FVCMin(c,t) | Capacity constraint lower bound | 8 |
| FVCMax(c,t) | Capacity constraint upper bound | 25 |
| FQBMin(u,q) | Blender quality lower bound | 9 |
| FQBMax(u,q) | Blender quality upper bound | 27 |
| FQVMin(u,m,q) | Batch quality lower bound | 6 |
| FQVMax(u,m,q) | Batch quality upper bound | 6 |

---

## BARON Solver Results

| Item | Value |
|------|-------|
| Best feasible solution | 34,167,967.96 (≈ 34.17 million) |
| Theoretical upper bound | 37,910,846.91 (≈ 37.91 million) |
| Optimality gap | 9.87% |
| Computation time | 18,000 seconds (5 hours; terminated at time limit) |
| Branch-and-bound iterations | 159,007 |
| Maximum nodes in memory | 68,259 |

---

## Notes on Porting to Python

Because `case1.gms` is in scalar form, there are two porting strategies.

**Approach A (Direct parsing of scalar form)**: Parse the 3,573 variable declarations, 3,428 constraint expressions, and variable bounds from `case1.gms` using regular expressions, then convert them directly into Pyomo variables and constraints. This reproduces the numerically identical problem without requiring semantic understanding.

**Approach B (Semantics-based reconstruction)**: Parse `all_sets.txt` and `all_parameters.txt`, then build an indexed Pyomo model based on the paper's mathematical formulation. Implement mass balance, quality tracking, yield, and capacity constraints individually. This provides deeper understanding but requires accurately interpreting the paper's formulation.

In either approach, the recommended solver progression is `IPOPT` (local optimum, free) → `SCIP` (global optimum, open-source).
