# Charlotte NPA Housing Affordability Analysis

**Course:** DTSC 2302 | UNC Charlotte  
**Contributors:** Brian Aguirre, Juan Thomas, Jonathan Obele, Maxwell Valentine
**Data Source:** Mecklenburg County Quality of Life Explorer  
**Live Map:** [Click here to view the interactive displacement risk map](https://juanthom99.github.io/Charlotte-NPA-Housing-Affordability-Analysis/outputs/displacement_risk_map.html)

---

## Project Overview

This project analyzes housing affordability across all 459 Neighborhood Profile Areas (NPAs) in Mecklenburg County, NC. Using 2023 data sourced from the Charlotte-Mecklenburg Quality of Life Explorer, we built two models: a regression model to predict median home sales price at the NPA level, and a classification model to identify which neighborhoods are at risk of housing-driven displacement. The outputs are stored in a SQLite database, reproduced through five structured Jupyter notebooks, and presented through a publicly accessible interactive map.

The project began with a straightforward question about predicting home values. It evolved into something more meaningful: a policy tool that identifies where rising prices are outpacing what residents can afford, providing city planners and community organizations with an actionable neighborhood-level triage framework.

---

## Live Interactive Map

The displacement risk map is publicly accessible and requires no software to use.

**URL:** https://juanthom99.github.io/Charlotte-NPA-Housing-Affordability-Analysis/outputs/displacement_risk_map.html

Each NPA polygon is colored red for At Risk or blue for Stable. Clicking any neighborhood opens a popup showing average home sales price, median household income, median rent, student absenteeism rate, bachelor's degree attainment, and the model's predicted displacement risk probability.

---

## Repository Structure
Charlotte-NPA-Housing-Affordability-Analysis/
├── data/
│   ├── charlotte_housing.db          # SQLite database (all raw + model tables)
│   ├── npa_boundaries.geojson        # NPA boundary geometry for mapping
│   └── raw/                          # Original CSVs from QoL Explorer
│       ├── Home_Sales_Price.csv
│       ├── Household_Income.csv
│       ├── Housing_Size.csv
│       ├── Rental_Costs.csv
│       ├── Education_Level_-Bachelor_s_Degree.csv
│       ├── Student_Absenteeism.csv
│       ├── Tree_Canopy.csv
│       ├── Water_Consumption.csv
│       └── Voter_Participation.csv
├── notebooks/
│   ├── 01_data_collection.ipynb      # Load CSVs into SQLite, merge by NPA ID
│   ├── 02_eda.ipynb                  # EDA, null handling, label engineering
│   ├── 03_regression.ipynb          # 7-model regression comparison + stability check
│   ├── 04_classification.ipynb       # 7-model classification comparison + stability check
│   └── 05_interactive_map.ipynb      # Folium map generation from model outputs
├── outputs/
│   └── displacement_risk_map.html    # Deployable interactive map
└── README.md---

## Data and Features

All data was sourced from the [Mecklenburg County Quality of Life Explorer](https://maps.mecklenburgcountync.gov/qol/). Each CSV was loaded into its own table in the SQLite database and joined by NPA ID into a single analysis-ready table.

| Feature | Description | Year |
|---|---|---|
| home_sales_price | Average home sales price per NPA (target) | 2023 |
| household_income | Median household income | 2023 |
| housing_size_sqft | Average heated sq ft of single-family homes | 2023 |
| median_rent | Median monthly rental cost | 2023 |
| bachelors_pct | % of adults 25+ with a Bachelor's degree | 2023 |
| absenteeism_pct | % of CMS students absent 10%+ of school days | 2023 |
| tree_canopy_pct | % of land covered by tree canopy | 2023 |
| water_consumption_gpd | Avg daily water use per single-family unit (gallons) | 2023 |
| voter_participation_pct | % of registered voters who voted in general election | 2025 |

**Feature decisions made during development:**

Public Health Insurance was originally proposed as a feature but was dropped because the most recent available data was from 2017, creating a six-year temporal gap with the 2023 target. It was replaced with Rental Costs, which captures a similar signal around economic precarity while remaining temporally aligned. Voter Participation is the only feature not from 2023. The 2025 general election data was used because it is the most recent available. This gap is documented in the database year columns and acknowledged as a limitation.

---

## Methodology

### Database Design

Raw CSVs were loaded into individual source tables in SQLite, then merged via LEFT JOINs on NPA ID into a single npa_features table. A cleaned, model-ready version (npa_features_model) is written back to the database by the EDA notebook and consumed by both modeling notebooks.

### Target Variable

**Regression target:** log_home_sales_price, the natural log of 2023 average home sales price. The raw distribution is heavily right-skewed. The log transform stabilizes variance and brings OLS residuals closer to normality.

**Classification target:** displacement_risk, a binary label engineered using the standard 30% housing affordability rule. For each NPA, we computed the implied monthly mortgage payment assuming a 30-year fixed loan at 7% APR and compared the annualized cost to 30% of median household income. NPAs where implied annual housing cost exceeds that threshold are labeled At Risk (1). All others are labeled Stable (0). This threshold is consistent with HUD affordability standards.

### Regression Model

Seven candidate models were evaluated using the same pipeline: 80/20 train/test split, 5-fold cross-validation on training data only, and a multi-seed stability check across 10 random states on the winner.

**Models evaluated:** OLS scaled (8 features), OLS reduced (5 features), Ridge with CV alpha, Lasso with CV alpha, KNN Regressor, Random Forest, Gradient Boosting.

**Key finding:** Mean test R² across 10 random splits was 0.06. NPA-level socioeconomic features carry limited signal for predicting exact home prices. However, applying the Fannie Mae 36% DTI affordability test to model-predicted prices correctly classified 77.5% of test NPAs by displacement risk, well above the 57.3% baseline of always predicting Stable.

### Classification Model

Seven candidate classifiers were evaluated using stratified 80/20 train/test splits and 5-fold stratified cross-validation. The winning model was subjected to a multi-seed stability check across 10 random states.

**Models evaluated:** Logistic Regression (unregularized), Logistic L1 (Lasso), Logistic L2 (Ridge), KNN (k=10), Decision Tree, Random Forest, Gradient Boosting.

**Key finding:** The winning classifier achieved 77.5% test accuracy, well above the 61% majority-class baseline.

### Ethical Design

Protected attributes such as race and ethnicity were never used as features. Voter participation was included as a civic engagement proxy with explicit documentation of its demographic sensitivity. All model outputs are framed as neighborhood-level policy tools, not judgments of individual residents. The displacement risk label is tied directly to HUD standards rather than an arbitrary statistical cutoff.

### Feature Leakage Controls

All feature lists are hardcoded. The following columns are explicitly forbidden from appearing in any feature set: home_sales_price, log_home_sales_price, cost_to_affordable_ratio, rent_to_income_ratio, income_per_sqft, displacement_risk. A runtime leakage check raises an error if any forbidden column appears in the feature list before training begins.

---

## Key Results

| Metric | Value |
|---|---|
| Regression test R² (mean, 10 seeds) | 0.06 |
| Displacement screening accuracy from regression | 77.5% |
| Classification test accuracy | 77.5% |
| Classification majority-class baseline | 61.0% |
| NPAs covered | 444 of 459 |

---

## Strengths

**Ethically grounded label design.** The displacement risk label is derived from the 30% affordability rule, a real policy standard used by HUD and city planners. The model's output can be directly interpreted and communicated to non-technical stakeholders without translation.

**Leakage-free architecture.** Feature lists are hardcoded. Derived columns are explicitly blocked before training. Imputation is fit on training data only and applied to the test set.

**Multi-seed stability verification.** Every winning model is subjected to a 10-seed stability check. In a dataset of 444 rows, a single train/test split can be misleading by 10 to 20 percentage points. Reporting mean and standard deviation across seeds gives an honest picture of model reliability.

**Reproducible data pipeline.** All raw data lives in version-controlled CSVs. The SQLite database is built deterministically from those CSVs by notebook 01. Any team member can clone the repo, run the notebooks in order, and reproduce every result exactly.

**Policy-facing deliverable.** The interactive map translates model outputs into a format a city planner can use immediately, without Python or any technical background. It is publicly accessible via GitHub Pages and includes contextual data in every popup.

**Transparent about limitations.** The regression model's low R² is reported honestly and addressed analytically by pivoting to the classification framework rather than overstating predictive power.

---

## Limitations

**Low regression R².** NPA-level socioeconomic aggregates are weak predictors of median sales price. Housing markets respond to supply, interest rates, buyer competition, and hyperlocal amenities not captured in the Quality of Life Explorer dataset.

**Cross-sectional data.** All features and the target are from a single year (2023). The model cannot identify trends, detect accelerating displacement, or respond to post-2023 market changes.

**Voter Participation temporal gap.** This feature is from 2025 while all others are from 2023. The gap is acknowledged and documented but is a genuine inconsistency.

**Median rent missing values.** Rental Costs had 52 null values out of 459 NPAs (approximately 11%). These were imputed using the training-set median.

**30-year fixed rate assumption.** The displacement risk label assumes a 30-year mortgage at 7% APR. In practice buyers use different loan products and rate environments vary over time.

**NPA-level aggregation.** All analysis is at the neighborhood level. The model cannot identify which residents within an At Risk NPA are most vulnerable. It is a triage tool, not a household-level risk assessment.

---

## Practical Uses

**Neighborhood prioritization.** A city planner with a fixed budget for housing intervention can use the At Risk classification to sort 459 NPAs into a priority queue, focusing resources where the affordability gap is largest.

**Pattern identification.** The interactive map makes spatial clustering of At Risk NPAs visible at a glance without running any analysis.

**Baseline for longitudinal tracking.** The 2023 snapshot can serve as a baseline. If QoL Explorer data is refreshed annually, the same pipeline can be rerun each year to track which NPAs have moved from Stable to At Risk, providing early warning of accelerating displacement.

**Supplementary evidence for policy.** The affordability ratio and displacement risk label are tied to HUD standards, making the model output directly citable in housing grant applications or policy proposals requiring evidence of community need.

**Extension with infrastructure data.** The map is designed to support a second data layer. Adding Charlotte's Capital Improvement Projects or building permit data as a GeoJSON overlay would allow planners to identify where At Risk NPAs overlap with new construction zones, the most direct evidence of displacement pressure available at the neighborhood level.

---

## How to Run

**Requirements:** Python 3.10+, Anaconda recommended.
