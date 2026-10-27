# Charlotte Housing Affordability Atlas

An exploratory analysis of neighborhood housing prices and an engineered affordability-stress measure in Mecklenburg County, North Carolina.

**[Explore the interactive map →](https://juanthom99.github.io/Charlotte-NPA-Housing-Affordability-Analysis/outputs/displacement_risk_map.html)** · [Analysis notebooks](notebooks/)

## Overview

This collaborative project joins public housing, economic, environmental, and civic indicators in SQLite, compares regression and classification models, and presents the analysis on an interactive neighborhood map.

**459 geographic areas are present in the source data and boundary file. The models use 444 records with usable home sales prices; 15 areas with missing prices are excluded from modeling.** The classification target is engineered from a mortgage-affordability scenario. It is not observed displacement or a prediction of who will move.

## Read the map

- **Above threshold:** estimated annual mortgage principal and interest exceeds 36% of median household income under the stated assumptions.
- **Within threshold:** principal and interest is at or below that scenario threshold. This does not establish affordability for every household.
- **No data:** no modeled label is available.

Colors show the **computed affordability label**, not classifier predictions. Popups show neighborhood context and a model probability only for held-out test records. Training records do not receive a held-out probability.

The map uses OpenStreetMap tiles with attribution and requires no API key. Its standalone HTML needs an internet connection for tiles and JavaScript libraries.

## Affordability scenario

The label assumes a **30-year fixed mortgage at 7% APR, a 20% down payment, and a 36% income threshold**. These are modeling assumptions, not a mortgage eligibility assessment.

```text
Loan principal = NPA home sales price × 0.80
Monthly principal and interest = amortization payment at 7% APR over 360 months
Stress label = annual principal and interest > median household income × 0.36
```

Taxes, insurance, maintenance, utilities, other debts, and differences between households are excluded. The legacy column `displacement_risk` stores this engineered label for compatibility; it should be interpreted only as affordability stress under this scenario.

## Analysis workflow

| Notebook | Purpose |
| --- | --- |
| [01 — Collection](notebooks/01_data_collection.ipynb) | Normalize nine CSV sources and join them by NPA ID in SQLite |
| [02 — EDA](notebooks/02_eda.ipynb) | Audit missing values, explore distributions, engineer labels, and save 444 model-ready records |
| [03 — Regression](notebooks/03_regression.ipynb) | Compare seven models for log home sales price and examine split stability |
| [04 — Classification](notebooks/04_classification.ipynb) | Compare seven classifiers for the engineered affordability label |
| [05 — Map](notebooks/05_interactive_map.ipynb) | Join labels and held-out predictions to the 459 neighborhood boundaries |

## Findings and interpretation

Exact home-price estimation is weak and unstable across random splits. Classification explores how neighborhood features relate to the engineered affordability measure; it does not validate a forecast of future displacement.

The notebooks include model comparisons and diagnostics. Scores are exploratory: candidate comparison reuses the test split, median imputation precedes internal cross-validation, and household income also participates in construction of the label. These limitations can make model scores optimistic.

## Data and limitations

Sources: [Charlotte-Mecklenburg Quality of Life Explorer](https://mcmap.org/qol/) and Mecklenburg County NPA boundary geometry. Housing and most contextual indicators use **2023** values; voter participation uses **2025** values. This is a mixed-year cross-sectional analysis, not a time-ordered forecast.

The eight model inputs are tree canopy, bachelor's degree attainment, water consumption, median rent, student absenteeism, household income, housing size, and voter participation. Neighborhood averages and medians do not describe every resident. Excluding race and ethnicity does not eliminate demographic proxies or unequal errors.

The map supports exploratory communication. It does not establish causality or support individual lending, housing, or benefit decisions.

## Reproduce the analysis

Use Python 3.10 or later in a dedicated environment:

```bash
python -m venv .venv
# Windows: .venv\Scripts\activate
# macOS / Linux: source .venv/bin/activate
python -m pip install -r requirements.txt
jupyter lab
```

Run notebooks **01 through 05** in order, from the repository root or `notebooks/`. Notebook 02 creates `npa_features_model`; notebooks 03 and 04 write model results; notebook 05 regenerates `outputs/displacement_risk_map.html`. The workflow updates the local SQLite database and exported map. Exact results can vary with package versions.

```text
data/raw/          Public CSV inputs
data/              SQLite database and NPA boundaries
notebooks/         Collection, analysis, modeling, and mapping
outputs/           Interactive map
requirements.txt   Python dependencies
```

## Collaboration

The original team contributors and commit history are preserved. This project demonstrates collaborative data integration, notebook development, exploratory modeling, and geospatial communication. Individual contributions should be described separately when presenting the work in an interview.
