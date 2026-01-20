[README.md](https://github.com/user-attachments/files/24731539/README.md)
# ECTO: Entropy-Initiated Coupled-Trait ODEs

**Computational Repository for PLOS ONE Submission**

[![Python 3.12+](https://img.shields.io/badge/python-3.12+-blue.svg)](https://www.python.org/downloads/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

---

## Overview

This repository contains the complete computational pipeline for the **Entropy-Initiated Coupled-Trait ODE (ECTO)** framework, a minimal, information-theoretic dynamical framework for modeling longitudinal cohort data. For each survey wave, item-level Likert responses are compressed into a normalized Shannon entropy index that summarizes cross-sectional dispersion; this index is used to initialize the low-dimensional state variables of an autonomous ODE system. ECTO then tracks interactions among a primary trait-like state, a secondary coupled state, and a latent environmental-stress component through phenomenological terms representing generic self-limitation, trade-offs, and feedback.

The framework is validated on two independent longitudinal datasets:
- **Primary Dataset (Track A):** Swedish Adoption/Twin Study of Aging (SATSA), 1984–2007 (6 waves)
- **Validation Dataset (Track B):** U.S. dental student longitudinal survey, D1–D4 (4 waves)

Entropy here functions as a compact summary of population heterogeneity rather than a dynamical driver, and the coupled ODEs supply an interpretable alternative to high-dimensional or black-box machine-learning approaches.

---

## Repository Structure

```
ECTO_PLOS1_repository2026.ipynb
│
├── TRACK A: SATSA Analysis (Modules A_0 through A_19)
│   ├── A_0:  Entropy extraction from raw Likert data
│   ├── A_1–A_4:  ODE system parameterization (manual & optimized)
│   ├── A_5:  Leave-One-Wave-Out cross-validation
│   ├── A_5.1: Glue cell for SATSA variable scope
│   ├── A_6:  Local sensitivity analysis (±10%)
│   ├── A_7:  Multistart robustness checks
│   ├── A_8:  Appendix figure generation
│   ├── A_9, A_9B: Entropy stability under simulated attrition (Appendix B)
│   ├── A_10, A_11: Null vs. ECTO model comparison (Appendix C.1)
│   ├── A_12: Global optimizer with tunable alpha
│   ├── A_13: Parameter sensitivity heatmaps (Appendix C.2)
│   └── A_15–A_19: Metabolic capacity sensitivity sweep (Appendix D)
│
├── TRACK B: Dental Student Validation (Modules B_0 through B_8)
│   ├── B_0:  Data extraction from original PLOS ONE supplement
│   ├── B_1:  Likert count computation from CSV
│   ├── B_2:  Entropy calculation with validation
│   ├── B_3:  ODE fitting (L-BFGS-B optimized)
│   ├── B_4:  Weakly optimized fit with E₀ and weighted SSE
│   ├── B_5:  Fully optimized fit with shape penalties
│   ├── B_6:  Leave-One-Wave-Out cross-validation
│   ├── B_7:  Local identifiability analysis (±10% sensitivity)
│   └── B_8:  Multistart optimization robustness
```

---

## Mathematical Framework

### Entropy Preprocessing

For each questionnaire item and survey wave, the categorical response distribution is summarized using Shannon entropy. Let **x** = [x₁, x₂, ..., xₙ] denote the raw Likert category counts for a given item in wave t, where n = 5 response categories.

These counts are converted into probabilities:

```
pᵢ = xᵢ / Σⱼ xⱼ
```

The Shannon entropy for that item and wave is:

```
H(t) = −Σᵢ pᵢ log₂(pᵢ)
```

Entropy is measured in bits and reflects dispersion of responses across categories. Each item entropy is normalized by its theoretical maximum:

```
H_norm(t) = H(t) / log₂(n) ∈ [0, 1]
```

### Core ODE System

The ECTO model consists of three coupled ordinary differential equations:

**State Variable N(t): Selection Pressure Dynamics**
```
dN/dt = μN − s(N)·N
```
where the selection pressure term is:
```
s(N) = αN + β²P
```

**State Variable P(t): Pleiotropic Dynamics**
```
dP/dt = μP − βP·(E_metabolic / G)
```
where the metabolic cost term is:
```
E_metabolic = c₁P + c₂N + c₃E_stress
```

**State Variable E_stress(t): Environmental Stress Dynamics**
```
dE_stress/dt = γE_stress·(N / (N + K))
```

The kernel N/(N+K) prevents unbounded amplification at low state values and introduces a natural saturation scale.

### Parameter Definitions

| Parameter | Description |
|-----------|-------------|
| **N(t)** | Primary entropy-initialized state variable |
| **P(t)** | Secondary coupled state variable |
| **E_stress(t)** | Auxiliary feedback state variable |
| **μ** | Baseline influx or persistence rate shared by N and P |
| **α** | Self-limiting constraint coefficient for N |
| **β** | Cross-state coupling coefficient (squared in N equation to ensure nonnegative damping) |
| **c₁, c₂, c₃** | Weights defining the metabolic cost term E_metabolic |
| **G** | Normalization constant setting the effective capacity scale |
| **γ** | Amplification rate for the stress state variable |
| **K** | Positive saturation constant controlling stress sensitivity |

All parameters are freely estimated during model fitting. They do not correspond to biological or psychological mechanisms; instead, they shape the qualitative behavior of the autonomous dynamical system.

### Initialization

All simulations are initialized by setting N(t₀) = H*(t₀), using the pooled entropy value from the first observed wave. The remaining state variables P(t₀) and E_stress(t₀) are initialized using small positive constants or fitted baseline values. After initialization, the system evolves autonomously with no additional inputs.

---

## Data Sources

### SATSA Dataset (Track A)
```
Pedersen, Nancy L. Swedish Adoption/Twin Study on Aging (SATSA), 
1984, 1987, 1990, 1993, 2004, 2007, and 2010. 
Inter-university Consortium for Political and Social Research [distributor], 2015-05-13.
https://doi.org/10.3886/ICPSR03843.v2
```

**Traits analyzed:** P9 Satisfaction, L10 Fulfillment, P4 Worry, P8 Hot-Tempered, P11 Indignant, L11 Depressed, N49 Curiosity, S28 Excitement Preference, I8 Impulsivity, A1 Competitive Ambition, L1 Life Optimism, P2 Rushed Feeling

**Flagship pair:** P8 Hot-Tempered (N) and P4 Worry (P)

### Dental Student Dataset (Track B)
```
Leite TC, Wankiiri-Hale CR, Shah NH, Vasquez CS, Pavlowski EM, Koury SE, et al. (2025) 
Change is never easy: Exploring the transition from undergraduate to dental student 
in a U.S.-based program. PLoS ONE 20(4): e0321494. 
https://doi.org/10.1371/journal.pone.0321494
```

**Variables analyzed:** 
- `supp`: Academic support perception
- `time`: Time management ability

**Entropy trajectories:**
- supp: 1.7278, 1.7198, 1.7755, 1.8126
- time: 1.4530, 1.6855, 1.8210, 1.7319

---

## Installation & Dependencies

### Requirements

```
Python >= 3.12
NumPy >= 1.26
SciPy >= 1.12
pandas >= 2.2
matplotlib >= 3.8
scikit-learn >= 1.4
seaborn >= 0.13 (for heatmap visualizations)
```

### Environment Setup

```bash
# Clone repository
git clone https://github.com/amr28693/ECTO_walkthrough_2026.git
cd ECTO_walkthrough_2026

# Create conda environment (recommended)
conda create -n ecto python=3.12
conda activate ecto

# Install dependencies
pip install numpy scipy pandas matplotlib scikit-learn seaborn

# Verify installation
python -c "import sys, numpy as np, scipy; print('Python', sys.version.split()[0], '| NumPy', np.__version__, '| SciPy', scipy.__version__)"
```

---

## Quick Start

### 1. Run Complete Pipeline

Open `ECTO_PLOS1_repository2026.ipynb` in Jupyter and execute cells sequentially. The notebook is designed for top-to-bottom execution.

### 2. Reproduce Key Results

**For SATSA trait coupling (Track A):**
```python
# Execute Modules A_0 through A_7
# Primary output: ODE fit metrics, LOO cross-validation results
```

**For dental student validation (Track B):**
```python
# Execute Modules B_0 through B_8
# Requires: Download pone.0321494.s005.xlsx from PLOS ONE supplement
# Place in: ./data/pone.0321494.s005.xlsx
```

### 3. Generate Supplementary Materials

| Supplementary Item | Module(s) |
|--------------------|-----------|
| Appendix B: Entropy Stability Under Simulated Attrition | A_9, A_9B |
| Appendix C.1: Null vs. ECTO Model Comparison | A_10, A_11 |
| Appendix C.2: Parameter Sensitivity Heatmaps | A_13 |
| Appendix D: Metabolic Capacity Sensitivity Sweep | A_15–A_19 |

---

## Key Results

### SATSA Full-Data Fit (Representative)

```
RMSE_N = 0.1541,  R²_N = 0.7525
RMSE_P = 0.1896,  R²_P = 0.7112
```

### SATSA Leave-One-Wave-Out Validation

```
LOO RMSE_N = 0.1995
LOO RMSE_P = 0.3056
```

### Dental Student Full Fit

```
RMSE_supp = 0.2249,  R²_supp = 0.6918
RMSE_time = 0.2406,  R²_time = 0.5759
```

### Dental Student LOO Validation

```
LOO RMSE_N = 0.2903
LOO RMSE_P = 0.2800
```

### Demonstrative Parameter Set (Set 3 from paper)

```
α = 0.098, μ = 0.00001, β = 0.17117, γ = 0.03
c₁ = 2.0, c₂ = 0.21, c₃ = 0.0, K = 0.5

State Variable (N): RMSE = 0.1552, R² = 0.7491, r = 0.879 (p = 0.021)
State Variable (P): RMSE = 0.2205, R² = 0.6095, r = 0.803 (p = 0.055)
```

---

## Validation Methods

### Leave-One-Wave-Out Cross-Validation

For each of the six (SATSA) or four (Dental) waves, the model is refit on the remaining timepoints, and the held-out wave is predicted from forward simulation.

### Local Sensitivity Analysis (±10%)

Parameters are perturbed ±10% from optimized values while recording changes in total RMSE. Results indicate which parameters are meaningfully constrained by the data versus weakly identifiable.

### Multistart Robustness

Multiple optimization runs from jittered initial conditions (±20%) verify convergence to consistent optima:

```
Multistart SSE — mean ≈ 0.488, sd ≈ 0.0001, min ≈ 0.4881, max ≈ 0.4885
```

The narrow range indicates a well-behaved optimization landscape.

---

## Metrics Reported

| Metric | Description |
|--------|-------------|
| **RMSE** | Root Mean Squared Error between model and empirical trajectories |
| **R²** | Coefficient of determination |
| **Pearson r** | Correlation coefficient with p-value |
| **DTW** | Dynamic Time Warping distance (shape similarity) |

---

## Extending the Framework

### Adding New Datasets

1. Format Likert data as frequency counts per response category per wave
2. Compute Shannon entropy: `H = −Σ pᵢ log₂(pᵢ)`
3. Normalize entropy series: `(x − min) / (max − min)`
4. Adapt ODE time axis to match wave spacing

### Modifying the ODE System

The core system is defined in the `system()` function:

```python
def system(y, t, mu, alpha, beta, gamma, c1, c2, c3, K):
    N, P, E = y
    E_dynamic = c1 * P + c2 * N + c3 * E
    dNdt = mu * N - alpha * N**2 - beta**2 * N * P
    dPdt = mu * P - beta * P * E_dynamic
    dEdt = gamma * E * (N / (N + K)) if (N + K) != 0 else 0.0
    return [dNdt, dPdt, dEdt]
```

---

## Troubleshooting

| Issue | Solution |
|-------|----------|
| `FileNotFoundError` for dental data | Download `pone.0321494.s005.xlsx` from [PLOS ONE](https://doi.org/10.1371/journal.pone.0321494.s005) and place in `./data/` |
| Optimizer converges to boundary | Widen parameter bounds or adjust initial guess |
| `P(t) = 0` invariant | Ensure `P_empirical[0] > 0`; add small epsilon (1e-6) to initial conditions |
| Numerical instability | Check for parameter combinations causing stiffness |

---

## Citation

If you use this code or methodology, please cite:

```bibtex
@article{Rodriguez2026,
  title={An Entropy-initiated Coupled-Trait ODE Framework for Modeling Longitudinal Cohort Dynamics},
  author={Rodriguez, Anderson M.},
  journal={PLOS ONE},
  year={2026},
  doi={[DOI]}
}
```

---

## License

This code is released under the MIT License. The underlying SATSA and dental student datasets are subject to their respective data use agreements (ICPSR and PLOS ONE).

---

## Contact

For questions regarding the code or methodology, please open an issue on this repository or contact: amr28693@uga.edu

---

## Acknowledgments

- SATSA data provided by ICPSR (Study #3843)
- Dental student data from Leite et al. (2025), PLOS ONE
- Computational framework developed using NumPy, SciPy, pandas, matplotlib, and scikit-learn
