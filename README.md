# Charlotte NPA Housing Affordability Analysis

**Course:** DTSC 2302 | UNC Charlotte
**Data Source:** Mecklenburg County Quality of Life Explorer
**Live Map:** [Click here to view the interactive displacement risk map](https://juanthom99.github.io/Charlotte-NPA-Housing-Affordability-Analysis/outputs/displacement_risk_map.html)

---

## Project Overview

This project analyzes housing affordability across all 459 Neighborhood Profile Areas (NPAs) in Mecklenburg County, NC. Using 2023 data sourced from the Charlotte-Mecklenburg Quality of Life Explorer, we built two models: a regression model to predict median home sales price at the NPA level, and a classification model to identify which neighborhoods are at risk of housing-driven displacement. The outputs are stored in a SQLite database, reproduced through five structured Jupyter notebooks, and presented through a publicly accessible interactive map.

The project began with a straightforward question about predicting home values. It evolved into something more meaningful: a policy tool that identifies where rising prices are outpacing what residents can afford, providing city planners and community organizations with an actionable neighborhood-level triage framework.

---

## Live Interactive Map

**URL:** https://juanthom99.github.io/Charlotte-NPA-Housing-Affordability-Analysis/outputs/displacement_risk_map.html

Each NPA polygon is colored red for At Risk or blue for Stable. Clicking any neighborhood opens a popup showing average home sales price, median household income, median rent, student absenteeism rate, bachelor's degree attainment, and the model's predicted displacement risk probability. The map was built using Folium and the official Mecklenburg County NPA boundary shapefile and requires no software to use.

---

## Repository Structure

Charlotte-NPA-Housing-Affordability-Analysis/
├── data/
│   ├── charlotte_housing.db
│   ├── npa_boundaries.geojson
│   └── raw/
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
│   ├── 01_data_collection.ipynb
│   ├── 02_eda.ipynb
│   ├── 03_regression.ipynb
│   ├── 04_classification.ipynb
│   └── 05_interactive_map.ipynb
├── outputs/
│   └── displacement_risk_map.html
└── README.md---

## Data and Features

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

Public Health Insurance was originally proposed as a feature but was dropped because the most recent available data was from 2017, creating a six-year temporal gap with the 2023 target. It was replaced with Rental Costs. Voter Participation is the only feature not from 2023. The 2025 general election data was used because it is the most recent available and civic engagement patterns are structurally stable across general elections of the same type.

---

## Methodology

### Database Design

Raw CSVs were loaded into individual source tables in SQLite, then merged via LEFT JOINs on NPA ID into a single npa_features table. A cleaned model-ready version called npa_features_model is written back to the database by the EDA notebook and consumed by both modeling notebooks.

### Target Variable

**Regression target:** log_home_sales_price, the natural log of 2023 average home sales price. The raw distribution is heavily right-skewed so the log transform stabilizes variance and brings OLS residuals closer to normality.

**Classification target:** displacement_risk, a binary label engineered using the standard 30% housing affordability rule. For each NPA, we computed the implied monthly mortgage payment assuming a 30-year fixed loan at 7% APR and compared the annualized cost to 30% of median household income. NPAs where implied annual housing cost exceeds that threshold are labeled At Risk (1). All others are labeled Stable (0). This threshold is consistent with HUD affordability standards.

### Regression Model

Seven candidate models were evaluated using an 80/20 train/test split, 5-fold cross-validation on training data only, and a multi-seed stability check across 10 random states on the winner. Models evaluated: OLS scaled (8 features), OLS reduced (5 features), Ridge with CV alpha, Lasso with CV alpha, KNN Regressor, Random Forest, Gradient Boosting.

The selected model was the scaled OLS using all eight features, achieving the highest single-split test R² of 0.2396. However the multi-seed stability check revealed significant variability, with a mean test R² of 0.062 and a standard deviation of 0.142 across 10 random splits. On the dollar scale the model produced a test RMSE of $194,441 and a mean absolute error of $124,116, approximately 32.9% of the median home price. The OLS model is not reliable for accurate price estimation.

### Classification Model

Seven candidate classifiers were evaluated using stratified 80/20 train/test splits and 5-fold stratified cross-validation. Models evaluated: Logistic Regression (unregularized), Logistic L1, Logistic L2, KNN (k=10), Decision Tree, Random Forest, Gradient Boosting.

The Random Forest classifier was selected as the final model. It achieved a test accuracy of 71.9% and a ROC AUC of 0.797, outperforming all other models and consistently exceeding the 61% majority-class baseline. A multi-seed stability check across 10 random splits confirmed reliability with a mean test accuracy of 0.749 and a standard deviation of 0.028.

### Ethical Design

Protected attributes such as race and ethnicity were never used as features. All model outputs are framed as neighborhood-level policy tools, not judgments of individual residents. The displacement risk label is tied directly to HUD standards rather than an arbitrary statistical cutoff.

### Feature Leakage Controls

All feature lists are hardcoded. The following columns are explicitly forbidden from appearing in any feature set: home_sales_price, log_home_sales_price, cost_to_affordable_ratio, rent_to_income_ratio, income_per_sqft, displacement_risk. A runtime leakage check raises an error if any forbidden column appears in the feature list before training begins.

---

## Key Results

| Metric | Value |
|---|---|
| Regression single-split test R² | 0.2396 |
| Regression mean test R² (10 seeds) | 0.062 |
| Regression test RMSE | $194,441 |
| Classification test accuracy | 71.9% |
| Classification ROC AUC | 0.797 |
| Classification stability mean (10 seeds) | 0.749 ± 0.028 |
| Classification majority-class baseline | 61.0% |
| NPAs covered | 444 of 459 |

---

## Strengths

**Ethically grounded label design.** The displacement risk label is derived from the 30% affordability rule, a real policy standard used by HUD and city planners. The model output can be directly interpreted and communicated to non-technical stakeholders without translation.

**Leakage-free architecture.** Feature lists are hardcoded. Derived columns are explicitly blocked before training. Imputation is fit on training data only and applied to the test set.

**Multi-seed stability verification.** Every winning model is subjected to a 10-seed stability check. Reporting mean and standard deviation across seeds gives an honest picture of model reliability rather than a lucky single split.

**Reproducible data pipeline.** All raw data lives in version-controlled CSVs. The SQLite database is built deterministically from those CSVs by notebook 01. Any team member can clone the repo, run the notebooks in order, and reproduce every result exactly.

**Policy-facing deliverable.** The interactive map translates model outputs into a format a city planner can use immediately without Python or any technical background. It is publicly accessible via GitHub Pages.

**Transparent about limitations.** The regression model's low mean R² is reported honestly and the project pivots to the classification framework rather than overstating predictive power.

---

## Limitations

**Unstable regression performance.** While the single-split test R² was 0.2396, the multi-seed mean dropped to 0.062 with high variance, indicating the model is not reliable for precise price prediction.

**Cross-sectional data.** All features and the target are from a single year (2023). The model cannot identify trends or detect accelerating displacement.

**Voter Participation temporal gap.** This feature is from 2025 while all others are from 2023. The gap is acknowledged and documented.

**Median rent missing values.** Rental Costs had 52 null values out of 459 NPAs (approximately 11%). These were imputed using the training-set median.

**30-year fixed rate assumption.** The displacement risk label assumes a 30-year mortgage at 7% APR. In practice buyers use different loan products and rate environments vary over time.

**NPA-level aggregation.** All analysis is at the neighborhood level. The model cannot identify which residents within an At Risk NPA are most vulnerable. It is a triage tool, not a household-level risk assessment.

---

## Practical Uses

**Neighborhood prioritization.** A city planner with a fixed budget for housing intervention can use the At Risk classification to sort NPAs into a priority queue, focusing resources where the affordability gap is largest.

**Pattern identification.** The interactive map makes spatial clustering of At Risk NPAs visible at a glance without running any analysis.

**Baseline for longitudinal tracking.** The 2023 snapshot can serve as a baseline. If QoL Explorer data is refreshed annually, the same pipeline can be rerun each year to track which NPAs have moved from Stable to At Risk, providing early warning of accelerating displacement.

**Supplementary evidence for policy.** The affordability ratio and displacement risk label are tied to HUD standards, making the model output directly citable in housing grant applications or policy proposals.

**Extension with infrastructure data.** The map is designed to support a second data layer. Adding Charlotte's Capital Improvement Projects or building permit data as a GeoJSON overlay would allow planners to identify where At Risk NPAs overlap with new construction zones.

---

## How to Run

Requirements: Python 3.10+, Anaconda recommended.
pip install pandas numpy matplotlib seaborn scikit-learn statsmodels folium geopandas pyogrio
Run notebooks in order: 01, 02, 03, 04, 05. The interactive map will be saved to outputs/displacement_risk_map.html. All notebooks after 01 use a dynamic base path that resolves correctly regardless of where the repo is cloned.

---

## Data Citation

All data sourced from the Charlotte-Mecklenburg Quality of Life Explorer, a partnership between UNC Charlotte Urban Institute, the City of Charlotte, and Mecklenburg County. NPA boundary geometry from Mecklenburg County GIS, QOL_NPA_2020_final_projected shapefile, converted to WGS84 GeoJSON. Affordability methodology based on HUD's 30% housing cost burden standard and Fannie Mae's 36% debt-to-income ratio guideline.
