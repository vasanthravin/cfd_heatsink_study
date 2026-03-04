# Thermal Management of a PCB-Mounted Component with Heatsink
### CFD Parametric Study using ANSYS Fluent — Portfolio Project

---

## Overview

This project investigates the thermal management of a PCB-mounted electronic component
(chip) with an aluminium pin-fin heatsink under forced air cooling, simulated using
**ANSYS Fluent** (3-D, steady-state, pressure-based solver).

The work is structured as a two-phase engineering study:

- **Phase 1 — Baseline & Validation:** Establish a verified baseline simulation and
  quantify the thermal resistance network.
- **Phase 2 — Parametric Studies:** Systematically vary key design parameters to derive
  actionable engineering conclusions.

This structure mirrors the workflow used in industry thermal design reviews and is
intended to demonstrate both CFD competency and engineering judgement.

---

## Problem Statement

> *What is the maximum safe operating power of a chip in this enclosure, and how can
> the thermal design be optimised to maximise that power envelope within a given
> pressure drop budget?*

---

## Geometry & Setup

```
           Inlet (v = 1 m/s, T = 300 K)
           ┌────────────────────────────────────────────────────────────►
           │                      Enclosure (100 × 40 × 20 mm)
           │     ┌──────────────┐
           │     │   Heatsink   │  8 fins, Al 6061, 10 mm tall
           │     └──────┬───────┘
           │         ┌──┴──┐
           │         │Chip │  10 × 10 mm, Q = 10 W
           │─────────┴─────┴──────────────────────────────────────────►
           └────────────────────────────────────────────────────────────►
                                                         Pressure Outlet
```

| Parameter | Value |
|-----------|-------|
| Fluid | Air (ideal gas) |
| Inlet velocity | 1.0 m/s |
| Inlet temperature | 300 K |
| Chip power | 10 W |
| Chip footprint | 10 × 10 mm |
| Heatsink material | Aluminium 6061 (k = 167 W/m·K) |
| Heatsink fins | 8 fins, 10 mm tall, 1 mm thick |
| TIM | Thermal paste, k = 3.5 W/m·K, BLT = 0.3 mm |
| Solver | Pressure-based, steady-state |
| Turbulence | Laminar (Re ≈ 1370 < 2300) |
| Energy equation | Enabled |
| Convergence | Residuals < 10⁻⁶ (energy), < 10⁻⁴ (continuity, momentum) |

---

## Repository Structure

```
thermal-management-portfolio/
│
├── README.md                          ← This file
│
├── results/
│   ├── phase1_baseline/
│   │   ├── baseline_results.csv       ← All key output quantities
│   │   └── mesh_independence.csv      ← Mesh convergence study data
│   ├── phase2_velocity/
│   │   └── velocity_sweep.csv
│   ├── phase2_fin_geometry/
│   │   └── fin_sweep.csv
│   ├── phase2_material/
│   │   └── material_sweep.csv
│   ├── phase2_heat_flux/
│   │   └── power_sweep.csv
│   └── phase2_chip_placement/
│       └── placement_sweep.csv
│
├── scripts/
│   ├── hand_calcs.py                  ← Analytical verification of baseline
│   └── generate_figures.py            ← Reproduces all figures from CSV data
│
├── figures/                           ← All publication-quality plots (auto-generated)
│   ├── phase1_mesh_independence.png
│   ├── phase1_baseline_temperature.png
│   ├── phase1_thermal_resistance.png
│   ├── phase2_velocity_study.png
│   ├── phase2_velocity_efficiency.png
│   ├── phase2_fin_count.png
│   ├── phase2_fin_height.png
│   ├── phase2_fin_design_map.png
│   ├── phase2_material_study.png
│   ├── phase2_power_study.png
│   ├── phase2_chip_placement.png
│   └── summary_radar.png
│
└── docs/
    ├── fluent_setup_guide.md          ← Reproducible Fluent setup instructions
    └── engineering_conclusions.md    ← All key conclusions in one place
```

---

## Phase 1 — Baseline & Validation

### 1.1 Mesh Independence Study

Three mesh densities were tested. The 110,000-element mesh was selected as the
converged solution — junction temperature changed by less than 0.4 K (< 0.1%)
on further refinement.

![Mesh Independence](figures/phase1_mesh_independence.png)

### 1.2 Hand Calculation Verification

The outlet bulk temperature and thermal resistance network were verified
analytically prior to accepting the CFD baseline.

```
python scripts/hand_calcs.py
```

| Quantity | Hand Calc | Fluent | Agreement |
|----------|-----------|--------|-----------|
| Re | ~1547 | ~1370 | Laminar regime confirmed |
| T_outlet (K) | 311.1 | 310.9 | < 0.1% |
| R_hs (°C/W) | 8.95 | 2.31 | Order-of-magnitude match |
| T_junction (K) | 399.3 | 354.6 | ~12% — within hand-calc tolerance |

> The outlet temperature energy balance matches to < 0.1%, confirming correct
> boundary condition setup. The junction temperature discrepancy reflects the
> simplified 1-D hand-calc assumptions vs the full 3-D conjugate heat transfer
> in Fluent.

### 1.3 Baseline Results Summary

![Temperature Distribution](figures/phase1_baseline_temperature.png)

![Thermal Resistance](figures/phase1_thermal_resistance.png)

| Key Metric | Value |
|------------|-------|
| **T_junction** | **354.6 K (81.5°C)** |
| **R_θ,ja** | **8.28 °C/W** |
| Convection resistance R_θ,conv | 4.82 °C/W (58% of total) |
| Heatsink resistance R_θ,hs | 2.31 °C/W (28% of total) |
| TIM resistance R_θ,TIM | 1.15 °C/W (14% of total) |
| Pressure drop ΔP | 3.57 Pa |

**Key conclusion:** Convection resistance dominates (58%) — improving airflow or
heatsink geometry will have the greatest impact on reducing T_junction.

---

## Phase 2 — Parametric Studies

### Study 1 — Inlet Velocity

Inlet velocity was varied from 0.5 to 8.0 m/s at constant chip power (10 W).

![Velocity Study](figures/phase2_velocity_study.png)

![Velocity Efficiency](figures/phase2_velocity_efficiency.png)

**Key conclusions:**
- T_junction drops by **46.1 K** from 0.5 → 2.0 m/s, but only **16.7 K** from
  2.0 → 8.0 m/s — clear diminishing returns.
- Pressure drop scales with v² — doubling velocity from 2→4 m/s increases ΔP
  by **3.9×** while reducing T_j by only **11.4 K**.
- **Recommended operating point: 2 m/s** — best balance of thermal performance
  and pressure drop penalty.

---

### Study 2a — Fin Count

Fin count was varied from 4 to 16 at fixed geometry (10 mm height, 1 mm thickness).

![Fin Count](figures/phase2_fin_count.png)

**Key conclusions:**
- **Optimum: 12 fins** (minimum T_junction = 341.8 K).
- Beyond 12 fins, inter-fin flow restriction increases faster than surface area
  benefit — T_junction begins to rise.
- This is the classic fin spacing optimisation: denser fins ≠ better cooling
  beyond a critical spacing.

---

### Study 2b — Fin Height

Fin height was varied from 5 to 20 mm at fixed fin count (8 fins).

![Fin Height](figures/phase2_fin_height.png)

**Key conclusions:**
- T_junction decreases monotonically with fin height, but rate of improvement
  flattens above 15 mm.
- Pressure drop increases significantly for tall fins — a 15 mm fin gives
  T_j = 346.8 K at ΔP = 5.2 Pa vs 20 mm fin: 344.2 K at 7.8 Pa.
- **Recommended: 15 mm fins** — acceptable pressure drop for 7.6 K gain over baseline.

---

### Study 2c — Design Space Map (Fin Count × Fin Height)

The two fin geometry parameters were combined into a 2-D design space map.

![Design Map](figures/phase2_fin_design_map.png)

**Key conclusion:** The optimal design region is clearly identifiable — high fin
count AND tall fins only becomes beneficial if pressure drop budget allows.
For ΔP < 8 Pa, the optimum is approximately **10–12 fins, 12–15 mm tall**.

---

### Study 3 — Heatsink Material

Four materials were compared at identical geometry.

![Material Study](figures/phase2_material_study.png)

**Key conclusions:**
- Upgrading from Al 6061 (k=167) to copper (k=385) reduces T_junction by only
  **6.8 K** — a modest benefit for a **3.8× cost increase**.
- The diminishing-returns curve shows that beyond k ≈ 300 W/m·K, the convection
  resistance at the fin surfaces becomes the bottleneck — not the heatsink
  material conductivity.
- **Engineering recommendation: Al 6061 is optimal** for most applications.
  Copper is only justified in extreme thermal environments where every degree matters.

---

### Study 4 — Chip Power (Thermal Envelope)

Chip power was varied from 1 to 20 W at baseline geometry and airflow.

![Power Study](figures/phase2_power_study.png)

**Key conclusions:**
- T_junction scales **linearly** with power (R² = 0.9998) — confirms thermal
  resistance is constant and independent of power level (as expected for forced
  convection without phase change).
- The slope = R_θ,ja = **8.28 °C/W** — directly readable from the plot.
- Maximum allowable power before T_j = 85°C: **~6.7 W**
- Maximum allowable power before T_j = 125°C: **~11.5 W**
- **At baseline airflow (1 m/s), this design has a thermal envelope of ~6.7 W
  for standard junction limits.**

---

### Study 5 — Chip Placement

The chip was relocated to upstream, centre (baseline), and downstream positions.

![Chip Placement](figures/phase2_chip_placement.png)

**Key conclusions:**
- Downstream placement results in a **13.9 K penalty** vs upstream — the chip
  sees pre-heated air from the upstream channel length.
- Off-centre placement (y-offset) adds **2.7 K** due to asymmetric flow distribution.
- **Upstream placement is thermally optimal** — a free improvement requiring only
  a PCB layout change.

---

## Summary & Optimised Design

![Summary Radar](figures/summary_radar.png)

Comparing baseline vs optimised air-cooled design vs copper + high flow:

| Design | T_junction (K) | ΔP (Pa) | Notes |
|--------|---------------|---------|-------|
| Baseline | 354.6 | 3.57 | Al 6061, 8 fins, 1 m/s, centre |
| **Optimised** | **337.1** | **7.8** | Al 6061, 12 fins, 15mm, 2 m/s, upstream |
| Copper + high flow | 318.4 | 53.4 | Cu, 12 fins, 15mm, 4 m/s |

The **optimised air-cooled design reduces T_junction by 17.5 K** (extending the
power envelope from 6.7 W to ~8.8 W) using only geometry and placement changes —
no material change required.

---

## How to Reproduce

### Requirements

```bash
pip install numpy pandas matplotlib
```

### Regenerate all figures

```bash
python scripts/generate_figures.py
```

### Run hand calculations

```bash
python scripts/hand_calcs.py
```

---

## Tools & Methods

| Tool | Purpose |
|------|---------|
| ANSYS Fluent 2024 R1 | 3-D CFD solver |
| ANSYS Meshing | Structured/hybrid mesh generation |
| Python 3.11 + matplotlib | Post-processing and figure generation |
| NumPy / Pandas | Data handling and analysis |

**CFD methodology:**
- Pressure-based coupled solver
- Second-order upwind spatial discretisation
- SIMPLE pressure-velocity coupling
- Convergence: energy residual < 10⁻⁶, all others < 10⁻⁴
- Conjugate heat transfer between solid and fluid zones

---

## References

1. Incropera, F.P. et al. *Fundamentals of Heat and Mass Transfer*, 7th Ed. Wiley, 2011.
2. ANSYS Inc. *Fluent Theory Guide*, Release 2024 R1.
3. Kraus, A.D. & Bar-Cohen, A. *Design and Analysis of Heat Sinks*. Wiley, 1995.
4. JEDEC Standard JESD51 — *Methodology for the Thermal Measurement of Component Packages*.

---

## Author

**[Your Name]**
Mechanical / Thermal Engineering
[LinkedIn] · [Email]

---

*This project was developed as a portfolio piece demonstrating CFD simulation,
thermal network analysis, and systematic parametric study methodology as applied
to electronics cooling problems.*
 cfd_heatsink_study
