# Charlotte NPA Housing Affordability Analysis

## Overview
This project provides a robust, data-driven "Triage Tool" to identify neighborhoods (NPAs) in Charlotte at risk of displacement. By shifting from noisy home-price prediction to direct binary classification, this analysis enables city planners to prioritize intervention areas based on stable demographic and economic signals.

## Ethical Standards & Responsible AI
We recognize that displacement risk modeling has significant social implications. To ensure our analysis is ethical and actionable:
* **Anti-Leakage Design:** We explicitly exclude variables that directly define the risk (e.g., `home_sales_price`), preventing the model from "cheating" by reverse-engineering the target.
* **Algorithmic Transparency:** By prioritizing interpretable models (Random Forest) and performing multi-seed stability testing, we ensure that our "At Risk" flags are based on stable patterns rather than statistical noise.
* **Focus on Triage, Not Targeting:** This tool is designed to assist policy makers in allocating resources to support vulnerable populations. We emphasize that these outputs should serve as a starting point for deeper qualitative community engagement, rather than a final verdict.

## Key Findings
* **Model Performance:** The **Random Forest Classifier** was identified as the most stable and accurate model, achieving a **mean test accuracy of 75%** (± 2.8% across 10 random seeds).
* **Primary Drivers:** The most critical predictors of displacement are **Household Income**, **Housing Size (sqft)**, and **Absenteeism rates**.
* **Robustness:** All results underwent rigorous multi-seed stability testing, ensuring that identified risk patterns are consistent across different data splits.

## Practical Application for Policy Makers
This tool is designed to support proactive, evidence-based urban planning in Charlotte through three key workflows:

Priority Triage: Planners can use the classification output as a "Priority List," filtering for NPAs flagged as "At Risk" (1) to focus limited intervention resources where they are most needed.

Decision Support: When evaluating new development permits, the model provides an objective risk-assessment layer. This data can justify conditions for approval, such as requiring higher percentages of affordable housing units in vulnerable neighborhoods.

Evidence-Based Advocacy: The analysis—specifically the Feature Importance findings—provides a rigorous data foundation for budget requests and federal grant applications (e.g., HUD funding), shifting arguments from subjective community observation to quantifiable structural risk.

## Repository Structure
* `notebooks/01_data_ingestion.ipynb`: Data collection and database construction.
* `notebooks/02_feature_engineering.ipynb`: Creation of the `displacement_risk` target and demographic feature set.
* `notebooks/03_regression_analysis.ipynb`: Initial price modeling and "thermometer" phase.
* `notebooks/04_classification.ipynb`: Final triage tool development, model comparison, and stability validation.
* `data/charlotte_housing.db`: Central SQL repository for data lineage and results.

## Acknowledgments
This analysis was developed to support evidence-based policy intervention strategies for Charlotte’s affordable housing initiatives, adhering to rigorous standards of transparency and algorithmic accountability.
