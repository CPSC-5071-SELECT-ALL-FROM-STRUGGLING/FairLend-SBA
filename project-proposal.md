**Equity in Small Business Lending:**

**Predicting Loan Default and Auditing Fairness in SBA-Guaranteed Loans**

CPSC 5071 – Data Management for Data Science – Winter 2026

**Team Members:** Bruna Porto and Jack Lichwa

# **1\. Problem Statement**

Small businesses account for roughly 44% of U.S. economic activity and create two-thirds of net new jobs, making them vital to economic growth \[1\]. Yet many small businesses fail within their first five years, often due to insufficient capital. The U.S. Small Business Administration (SBA) addresses this by guaranteeing loans to reduce lender risk—but when a guaranteed loan defaults, the financial loss is shared between the lending bank and U.S. taxpayers through the SBA. Between 1987 and 2014, approximately 17.6% of SBA-guaranteed loans were charged off \[2\]. For lenders, accurately predicting which loans will default is essential for managing portfolio risk, setting appropriate loan terms, and making sound approval decisions. For the SBA and policymakers, understanding the drivers of default can inform program design and resource allocation.

However, prediction alone is not sufficient. Research has documented persistent disparities in small business lending: women-owned and minority-owned businesses tend to receive smaller loans, face higher denial rates, and report lower satisfaction with lenders, even after accounting for firm characteristics \[3\]. Federal Reserve survey data confirm that these gaps persist, with minority-owned firms more likely to be denied credit or discouraged from applying \[4\]. As financial institutions increasingly adopt data-driven decision-making tools, a critical question emerges: could predictive models used in lending inadvertently encode or amplify these existing disparities? Building a default prediction model without examining its fairness implications risks automating inequality.

This project addresses two interconnected research questions:

• 	**Predictive:** Can we accurately predict whether an SBA-guaranteed loan will default based on loan characteristics, business attributes, and macroeconomic conditions at the time of origination? A reliable predictive model would enable lenders to better assess risk at the point of application, potentially reducing losses for both banks and the SBA while improving capital allocation to viable businesses.

• 	**Inferential:** After controlling for business fundamentals, do loan outcomes differ systematically across industry-state segments with varying concentrations of women-owned and minority-owned businesses? If a predictive default model were deployed in a lending context, would it produce equitable error rates across these demographic segments?

# **2\. Datasets**

The project integrates data at three levels of analysis—micro (loan-level), meso (industry demographics), and macro (economic conditions)—plus a validation source.

| Source | Format | Granularity | Key Variables | Join Strategy |
| :---- | :---- | :---- | :---- | :---- |
| SBA Loan Data (Kaggle) \[2\] | CSV (public repository) | Individual loan (\~900K records, 1987–2014) | Loan amount, term, NAICS sector, state, \# employees, new/existing, franchise, revolving credit, default status (MIS\_Status) | Primary dataset; join to others via NAICS sector \+ State \+ Year |
| Census Annual Business Survey \[5\] | JSON (REST API) | Industry × State aggregate | Firm count, receipts, employment by owner sex, race/ethnicity, veteran status, NAICS sector | Compute pct\_women\_owned, pct\_minority\_owned per NAICS × State; join to SBA loans |
| FRED Economic Data (St. Louis Fed) \[6\] | JSON (REST API) | Monthly time series | Federal Funds Rate (FEDFUNDS), Prime Rate (PRIME), CPI (CUUR0000SA0), Unemployment (UNRATE) | Join to SBA loans by approval year/month as macroeconomic controls |
| Fed Small Business Credit Survey \[4\] | PDF reports \+ chartbook data | Firm-level survey (national sample, \~7,600 firms) | Loan approval/denial by owner race & gender, credit scores, profitability, financial challenges | Validation reference: compare ecological patterns from SBA+ABS against firm-level SBCS findings |

**Preprocessing needs:** The SBA dataset requires cleaning of currency fields (removing “$” and commas), handling missing values in MIS\_Status and NAICS, and deriving the 2-digit industry sector code. Census ABS data will be queried via the Census API and requires alignment of NAICS code vintages. FRED data requires temporal alignment to loan approval dates. All sources will be loaded into a relational database (MySQL or SQLite) with a normalized schema.

# **3\. Algorithms and Technologies**

## **3.1 Data Management**

MySQL or SQLite for relational storage. We will design an ER model with entities for Loan, IndustrySector, DemographicProfile, and EconomicIndicator, and use SQL for exploratory queries, joins, and aggregations.

## **3.2 Predictive Modeling (scikit-learn)**

We will train and compare three classifiers to predict loan default (MIS\_Status \= CHGOFF): Logistic Regression (interpretable baseline with coefficient analysis), Random Forest, and HistGradientBoostingClassifier (handles missing values natively and scales to \~900K rows). Evaluation will use stratified k-fold cross-validation with AUC-ROC, precision, recall, and F1. We will also use threshold tuning (TunedThresholdClassifierCV) to explore how different decision thresholds affect lending decisions.

## **3.3 Inferential Analysis: Ecological Approach to Demographic Disparities**

The SBA loan dataset does not contain individual borrower demographics (gender, race). To address the inferential question, we adopt an ecological analysis approach using the Census ABS as a demographic overlay:

1\.   **Step 1 – Compute demographic composition:** Using Census ABS API data, we compute the percentage of women-owned firms (pct\_women\_owned) and minority-owned firms (pct\_minority\_owned) for each 2-digit NAICS sector × State combination.

2\.   **Step 2 – Join to SBA loans:** Each SBA loan record is enriched with these demographic composition percentages by matching on NAICS sector and State. This creates segment-level demographic context for each loan.

3\.   **Step 3 – Statistical testing:** We run logistic regression predicting default where pct\_women\_owned and pct\_minority\_owned are included as covariates alongside loan amount, term, employees, new/existing status, and macroeconomic indicators (from FRED). We test whether the demographic composition coefficients are statistically significant, which would indicate that loan outcomes vary by the demographic makeup of the industry-state segment even after controlling for business fundamentals.

4\.   **Step 4 – Fairness audit:** We split the test set into subgroups based on demographic composition (e.g., “high women-ownership segments” vs. “low women-ownership segments” using median splits) and compare the predictive model’s false positive rate and false negative rate across these groups. If the model disproportionately flags loans as risky in high-women-ownership or high-minority-ownership segments, this constitutes a fairness concern \[7\].

## **3.4 Validation via Fed Small Business Credit Survey**

Because our primary analysis is ecological (segment-level, not individual-level), we will use the Federal Reserve’s Small Business Credit Survey (SBCS) \[4\] as an independent validation reference. The SBCS contains firm-level data on owner demographics (gender, race, veteran status) alongside financing outcomes (approval/denial rates, lender satisfaction, credit challenges). Although the SBCS is a convenience sample and cannot be directly joined to our SBA loan data, it allows us to check whether the patterns we observe at the ecological level are *directionally consistent* with firm-level evidence. For example, if our ecological analysis finds that segments with higher minority business ownership have higher default rates, the SBCS can confirm whether minority-owned firms also report higher rates of financial challenges and credit denials at the individual level. This triangulation strengthens the credibility of our findings and helps guard against ecological fallacy.

# **4\. Risks**

• 	**Ecological inference limitation:** Census ABS data is aggregate (industry × state), so we cannot link individual loan outcomes to individual owner demographics. Our analysis identifies segment-level patterns, not individual-level causal claims. We will clearly frame this limitation and use the SBCS validation to mitigate it.

• 	**Temporal mismatch:** The SBA Kaggle dataset covers 1987–2014, while Census ABS data begins in 2017\. We may supplement with the older Survey of Business Owners (SBO, 2012\) \[9\] for better temporal overlap, or scope the analysis to the most recent SBA data subset.

• 	**Class imbalance:** Approximately 17.6% of SBA loans default. We will address this via stratified sampling, threshold tuning, and evaluation metrics robust to imbalance (AUC, F1).

• 	**SBCS sample limitations:** The SBCS is a convenience sample (\~7,600 firms), not a census. Its findings serve as directional validation, not definitive confirmation of our ecological analysis.

# **5\. Challenges**

The unique challenge of this project lies in the **multi-source data integration and the methodological care required for fairness analysis**. Joining loan-level microdata with aggregate demographic data and time-series macroeconomic data across different schemas, formats (CSV, JSON API), and time periods requires careful database design and validation. Additionally, translating an ecological fairness audit into well-scoped conclusions—without overstating segment-level correlations as individual-level evidence—demands both technical rigor and careful communication. The SBCS validation step adds methodological depth but also introduces complexity in synthesizing findings across different data sources with different sampling methodologies.

# **6\. References**

\[1\] U.S. Small Business Administration. “SBA Overview and History.” https://www.sba.gov/about-sba

\[2\] Li, M., Mickel, A., and Taylor, S. “Should This Loan be Approved or Denied?: A Large Dataset with Class Assignment Guidelines.” *Journal of Statistics Education*, 26(1), 2018\. Dataset: https://www.kaggle.com/datasets/mirbektoktogaraev/should-this-loan-be-approved-or-denied

\[3\] Bates, T. and Robb, A. “Disparities in Capital Access between Minority and Non-Minority-Owned Businesses.” *The ANNALS of the American Academy of Political and Social Science*, 613(1), 2007\.

\[4\] Federal Reserve Banks. “2025 Report on Employer Firms: Findings from the 2024 Small Business Credit Survey.” https://www.fedsmallbusiness.org/reports/survey/2025/2025-report-on-employer-firms

\[5\] U.S. Census Bureau. “Annual Business Survey (ABS).” https://www.census.gov/data/developers/data-sets/abs.html

\[6\] Federal Reserve Bank of St. Louis. “FRED Economic Data.” https://fred.stlouisfed.org/docs/api/fred/

\[7\] Barocas, S., Hardt, M., and Narayanan, A. *Fairness and Machine Learning: Limitations and Opportunities*. MIT Press, 2023\.

\[8\] Schweitzer, M. E. and Meyer, B. “Equal Access to Small-Business Credit.” Federal Reserve Bank of Cleveland, 2022\.

\[9\] U.S. Census Bureau. “Survey of Business Owners (SBO), 2012.” https://www.census.gov/programs-surveys/sbo.html

