# FairLend-SBA: Equitable Loan Default Prediction

## Overview
This repository contains the code and analysis for **FairLend-SBA**, a machine learning project focused on predicting Small Business Administration (SBA) loan defaults while actively mitigating algorithmic bias.

Historically, between 1987 and 2014, approximately 17.6% of SBA-guaranteed loans were charged off. While accurate risk assessment is critical, there are persistent disparities in lending to women-owned and minority-owned businesses. This project aims to build a reliable predictive model that ensures risk assessment algorithms do not amplify these existing structural inequalities.

## Research Questions
1. **Loan Default Prediction:** How accurately can we predict whether an SBA-guaranteed loan will default based on historical data?
2. **Disparity Analysis:** Do loan outcomes and model predictions differ systematically across industry-state segments that have varying concentrations of women-owned and minority-owned businesses?

## Dataset
The analysis integrates multiple external data sources:
* **SBA Loan Data (Kaggle):** Historical SBA loan records (1987–2014) with borrower details, loan terms, NAICS sector, state, and loan status (Paid in Full vs. Charged Off).
* **Census Annual Business Survey (ABS):** Industry × state aggregates on firm ownership by gender, race/ethnicity, and veteran status, accessed via the Census API.
* **FRED Economic Data (St. Louis Fed):** Macroeconomic indicators (Federal Funds Rate, Prime Rate, CPI, Unemployment) via REST API, aligned to loan approval dates.
* **Fed Small Business Credit Survey:** Firm-level survey data for validation, including owner demographics and financing outcomes.

All sources are joined using NAICS sector, state, and year/month as appropriate. Data is stored in a relational database (MySQL or SQLite) with a normalized schema.

## Project Structure
* `data/`: Raw and processed SBA datasets.
* `notebooks/`: Jupyter notebooks for exploratory data analysis (EDA), model training, and fairness evaluation.
* `reports/`: Final project report and presentation slides.

## Methodology
* **Data Preprocessing:** Handling missing values, encoding categorical variables, and normalizing financial figures.
* **Modeling:** Training classification algorithms to predict the probability of a loan being charged off.
* **Equality Auditing:** Evaluating models using metrics across different demographic and geographic segments to ensure equitable risk assessment.

## External Sources and Citations

* **SBA Loan Data (Kaggle):** https://www.kaggle.com/datasets/mirbektoktogaraev/should-this-loan-be-approved-or-denied
* **Census Annual Business Survey (ABS):** https://www.census.gov/data/developers/data-sets/abs.html
* **FRED Economic Data (St. Louis Fed):** https://fred.stlouisfed.org/docs/api/fred/
* **Fed Small Business Credit Survey:** https://www.fedsmallbusiness.org/reports/survey/2025/2025-report-on-employer-firms

References:
* Barocas, S., Hardt, M., and Narayanan, A. Fairness and Machine Learning: Limitations and Opportunities. MIT Press, 2023.
* Bates, T. and Robb, A. “Disparities in Capital Access between Minority and Non-Minority-Owned Businesses.” The ANNALS of the American Academy of Political and Social Science, 613(1), 2007.
* Li, M., Mickel, A., and Taylor, S. “Should This Loan be Approved or Denied?: A Large Dataset with Class Assignment Guidelines.” Journal of Statistics Education, 26(1), 2018.
* Schweitzer, M. E. and Meyer, B. “Equal Access to Small-Business Credit.” Federal Reserve Bank of Cleveland, 2022.
* U.S. Census Bureau. “Survey of Business Owners (SBO), 2012.” https://www.census.gov/programs-surveys/sbo.html
