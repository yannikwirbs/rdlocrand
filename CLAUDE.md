# CLAUDE.md — AI Assistant Guide for rdlocrand

## Overview

**rdlocrand** is a multi-language statistical software package implementing regression discontinuity (RD) designs under local randomization. It provides **parallel implementations in Python, R, and Stata** with consistent APIs, function names, and parameter conventions.

**Package purpose:** Provide methods for analyzing RD designs by treating units near the cutoff as if they were randomly assigned, including hypothesis testing, window selection, sensitivity analysis, and Rosenbaum bounds.

---

## Repository Structure

```
rdlocrand/
├── Python/rdlocrand/          # Python package (pip installable)
│   ├── pyproject.toml         # Build configuration
│   ├── setup.cfg              # Package metadata and dependencies
│   ├── ToBuild.txt            # Manual build/publish instructions
│   └── rdlocrand/src/rdlocrand/
│       ├── __init__.py        # Exports: rdrandinf, rdwinselect, rdsensitivity, rdrbounds
│       ├── rdlocrand_fun.py   # Shared helper/utility functions
│       ├── rdrandinf.py       # Randomization inference (main module)
│       ├── rdwinselect.py     # Window selection procedure
│       ├── rdsensitivity.py   # Sensitivity analysis
│       └── rdrbounds.py       # Rosenbaum bounds
├── R/rdlocrand/               # R package (CRAN publishable)
│   ├── DESCRIPTION            # Package metadata and dependencies
│   ├── NAMESPACE              # Exports (via Roxygen2)
│   ├── rdlocrand.Rproj        # RStudio project
│   ├── R/
│   │   ├── rdlocrand_package.R  # Package-level docs and imports
│   │   ├── rdlocrand_fun.R      # Shared helper functions
│   │   ├── rdrandinf.R
│   │   ├── rdwinselect.R
│   │   ├── rdsensitivity.R
│   │   └── rdrbounds.R
│   └── man/                   # Auto-generated Roxygen2 .Rd docs
├── stata/                     # Stata package (SSC/net installable)
│   ├── rdrandinf.ado
│   ├── rdwinselect.ado
│   ├── rdsensitivity.ado
│   ├── rdrbounds.ado
│   ├── rdrandinf_model.ado    # Helper for model specification
│   ├── rdwinselect_allcovs.ado # Helper for covariate processing
│   ├── rdlocrand_functions.do  # Mata source code
│   ├── *.mo                   # Compiled Mata binaries
│   ├── *.sthlp                # Stata help files
│   ├── *.pdf                  # PDF manuals (one per function)
│   ├── rdlocrand.pkg          # Package manifest
│   └── rdlocrand_illustration.do
├── README.md                  # Main project documentation
└── LICENSE.md                 # GPL-3.0
```

Each language directory also contains:
- `rdlocrand_illustration.*` — example/demo script
- `rdlocrand_senate.csv` / `.dta` — sample US Senate dataset

---

## Core Functions

All four functions exist in all three languages with consistent names:

| Function | Purpose |
|---|---|
| `rdrandinf` | Hypothesis testing via randomization inference; main estimation function |
| `rdwinselect` | Automated window selection for local randomization |
| `rdsensitivity` | Sensitivity analysis across window sizes and null hypotheses |
| `rdrbounds` | Rosenbaum bounds for unobserved confounders |

---

## Cross-Language Parameter Conventions

When adding or modifying parameters, maintain consistency across all three implementations:

| Parameter | Type | Description |
|---|---|---|
| `Y` | numeric vector | Outcome variable |
| `R` | numeric vector | Running variable (assignment variable) |
| `cutoff` | scalar | RD cutoff threshold (default: 0) |
| `wl`, `wr` | scalar | Left/right window boundaries |
| `statistic` | string | Test statistic: `'diffmeans'`, `'ksmirnov'`, `'ranksum'`, `'ttest'` |
| `p` | integer | Polynomial order for outcome adjustment |
| `kernel` | string | Weighting: `'uniform'`, `'triangular'`, `'epan'` |
| `reps` | integer | Monte Carlo replications for randomization inference |
| `seed` | integer | Random seed for reproducibility |
| `fuzzy` | vector | Fuzzy RD treatment indicator |
| `ci` | scalar | Confidence level or vector for CI inversion |
| `interfci` | scalar | Interference-robust CI |
| `obsmin`, `wobs` | integer | Minimum observations / observation step size |
| `wmin`, `wstep` | scalar | Minimum window / window step size |
| `nwindows` | integer | Number of windows to evaluate |
| `covs` | matrix/dataframe | Pre-determined covariates for balance testing |
| `covstype` | string | Covariate test type |
| `plot` | boolean | Whether to generate plots |
| `quietly` | boolean | Suppress output |

---

## Development Workflows

### Python

```bash
# Build the package
cd Python/rdlocrand
python3 -m build

# Install locally for testing
pip install ./dist/rdlocrand-*.tar.gz

# Run the illustration script
python3 rdlocrand_illustration.py

# Publish to PyPI (maintainers only)
twine upload dist/*
```

**Python version requirement:** >= 3.8

**Key dependencies** (from `setup.cfg`):
- `numpy >= 2.1.0`
- `pandas >= 2.2.0`
- `scipy >= 1.12.0`
- `statsmodels >= 0.14.1`
- `linearmodels >= 5.4`

### R

```r
# Install dependencies
install.packages(c("AER", "sandwich"))

# Generate documentation (requires devtools/Roxygen2)
devtools::document()

# Build and check package
R CMD build R/rdlocrand
R CMD check rdlocrand_*.tar.gz

# Install locally
install.packages("R/rdlocrand", repos = NULL, type = "source")

# Run illustration
source("R/rdlocrand/rdlocrand_illustration.R")
```

**R version requirement:** >= 3.1

**Key dependencies:** `AER`, `sandwich`

### Stata

```stata
* Install from GitHub (users)
net install rdlocrand, from(https://raw.githubusercontent.com/rdpackages/rdlocrand/master/stata) replace

* Recompile Mata code (if modifying .do file)
do rdlocrand_functions.do

* Run illustration
do rdlocrand_illustration.do
```

**Stata version requirement:** 13+

**Mata `.mo` files** are pre-compiled binaries. Recompile by running `rdlocrand_functions.do` in Stata. Do not manually edit `.mo` files.

---

## Code Conventions

### Naming
- All public functions use the `rd*` prefix: `rdrandinf`, `rdwinselect`, `rdsensitivity`, `rdrbounds`
- Helper/internal functions live in `rdlocrand_fun.*` files
- Internal utilities are not exported

### Documentation Style
- **Python:** Google/NumPy-style docstrings with Parameters, Returns, and References sections
- **R:** Roxygen2 (`#'`) format; docs auto-generated via `devtools::document()`
- **Stata:** `.sthlp` help files and separate `.pdf` manuals per function

### Statistical Design Patterns
- All functions support both sharp and fuzzy RD designs
- Test statistics are string-specified for portability across languages
- Random seed handling must be explicitly set for reproducible inference
- Confidence intervals are computed via test inversion (not formula-based)
- Window selection uses a symmetric-first, then asymmetric search

### Cross-language Parity
When adding a feature or fixing a bug in one language, **always implement the same change in all three languages** unless the change is truly language-specific (e.g., a Python-only packaging issue).

---

## No Automated Tests

There is **no dedicated test suite** in this repository. Validation relies on:
- The `rdlocrand_illustration.*` scripts (manual smoke tests)
- Cross-verification against published paper results
- Comparison across the three language implementations

When making changes, run the illustration scripts for the affected language(s) and verify output is consistent with prior results.

---

## Package Versioning

- **Python:** Version in `Python/rdlocrand/setup.cfg` (`version = 1.0.5`)
- **R:** Version in `R/rdlocrand/DESCRIPTION` (`Version: 1.1`)
- **Stata:** Distribution date in `stata/rdlocrand.pkg`

Versions across languages are **not synchronized** — each follows its own release cycle on its respective registry (PyPI, CRAN, SSC).

---

## Sample Data

`rdlocrand_senate.csv` / `.dta` — US Senate election data used for all examples:
- Running variable: margin of victory in previous election
- Outcome: vote share in next election
- Pre-determined covariates: various senator/state characteristics
- Cutoff: 0 (win vs. loss threshold)

---

## Key Academic References

Changes to statistical methodology should be consistent with:

1. Cattaneo, Frandsen, Titiunik (2015). "Randomization Inference in the Regression Discontinuity Design: An Application to Party Advantages in the U.S. Senate." *Journal of Causal Inference* 3(1).
2. Cattaneo, Titiunik, Vazquez-Bare (2016). "Inference in Regression Discontinuity Designs under Local Randomization." *Stata Journal* 16(2).
3. Cattaneo, Titiunik, Vazquez-Bare (2017). "Comparing Inference Approaches for RD Designs." *Journal of Policy Analysis and Management* 36(3).
4. Rosenbaum (2002). *Observational Studies*. Springer. (for `rdrbounds`)

---

## Distribution Channels

| Language | Registry | Install Command |
|---|---|---|
| Python | PyPI | `pip install rdlocrand` |
| R | CRAN | `install.packages('rdlocrand')` |
| Stata | SSC/GitHub | `net install rdlocrand, from(...) replace` |

**Project website:** https://rdpackages.github.io/rdlocrand
**Contact:** rdpackages@googlegroups.com
