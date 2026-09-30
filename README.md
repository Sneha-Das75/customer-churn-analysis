# Customer Churn Analysis (Python, SQLite, pandas)

An exploratory customer churn analysis using a small subscription dataset. The project examines which customers churned, how customer characteristics and support interactions relate to churn, and when cancellations occurred. Data is read from a SQLite database, cleaned and joined with pandas, then analysed using KPIs, grouped summaries, visualisations and correlation analysis.

> **About this project:** this is a **guided learning project**. The business problem, churn definition and database structure came from the guided material I followed; I worked through the cleaning, feature engineering, analysis and visualisation in a Jupyter notebook and wrote up the findings in an insights document. It is not an independently designed case study, and the dataset is small (21 customers), so the results are best read as practice findings rather than production-grade conclusions.

---

## Table of Contents
- [Objective](#objective)
- [Dataset](#dataset)
- [Tools](#tools)
- [Methodology](#methodology)
- [Analysis Performed](#analysis-performed)
- [Key Insights](#key-insights)
- [Limitations and Known Issues](#limitations-and-known-issues)
- [Project Files](#project-files)
- [How to Run](#how-to-run)
- [Skills Practiced](#skills-practiced)

---

## Objective

Follow the standard churn-analysis framing of **Who, Why, When**:

| Question | What the project looks at |
|---|---|
| **Who** left? | Churn by plan type, acquisition channel (subscription type) and state |
| **Why** did they leave? | Support escalations, contract type, plan type and the churn-risk score, using correlations |
| **When** did they leave? | Monthly trend of cancellations |

**Churn definition used (SaaS):** a customer has churned when their subscription is cancelled (`cancellation_date` is present, `churn_flag = 1`).

## Dataset

SQLite database `customer_churn.db` with three tables linked on `customerid`:

| Table | Rows | Columns |
|---|---|---|
| `db_customer` | 21 | customerid, name, country, state, gender, dob, interests, pincode |
| `db_subscription` | 21 | customerid, subscription_start_date, subscription_type, renewal_date, plan_type, contract_type, cancellation_date, cancellation_reason, monthly_charges, cltv, churn_score |
| `db_support` | 9 | customerid, complaint_date, escalations, csat_score, col_1, comment |

After cleaning and joining, the analysis table has **21 rows** (one per customer). Two columns were then added (`tenure_days`, `churn_risk`), giving 23 columns.

## Tools

- **Python** with **pandas**, **NumPy**
- **Matplotlib** and **Seaborn** for charts
- **SQLite** (`sqlite3`) to read the database into pandas
- **Jupyter Notebook** for the analysis

## Methodology

**1. Load data from SQL.** Connected to `customer_churn.db`, listed the tables from `sqlite_master`, and loaded each one into its own DataFrame.

**2. Clean each table.**
- `db_customer`: renamed `name` to `customer_name`; dropped `interests` and `pincode`; converted `dob` to datetime; standardised gender (`Men`/`Women` to `Male`/`Female`); filled 3 missing `country` values using a state-to-country mapping.
- `db_subscription`: converted `subscription_start_date`, `renewal_date` and `cancellation_date` to datetime.
- `db_support`: dropped `col_1` and `comment`; converted `complaint_date` to datetime.

**3. Engineer features.**
- `churn_flag`: 1 if a cancellation date exists, otherwise 0.
- `count`: number of complaints per customer.
- `tenure_days`: cancellation date minus start date for churned customers; today minus start date for active customers.
- `churn_risk`: banded from `churn_score` (above 70 = high, above 50 up to 70 = mid, 50 or below = low).
- `escalations` converted from Y/N to 1/0 for correlation.

**4. Join the tables.** Left-joined subscription, customer and support on `customerid`. The first join produced 23 rows because some customers had multiple complaints, so the support table was reduced to one row per customer (9 rows to 7) before re-joining, giving a 21 x 21 table.

**5. Analyse and visualise.** Calculated KPIs and grouped summaries, then used charts, a correlation heatmap, pairplot, catplot and pivot tables to explore churn patterns.

## Analysis Performed

**KPIs**
- Churn rate and retention rate
- ARPU (average revenue per user)
- Average customer tenure
- Revenue lost to churn (sum of monthly charges for churned customers)
- Escalation rate and its correlation with churn

**Segment breakdowns**
- Churn rate by plan type
- Churn rate, total revenue and user count by state
- Churn rate, total revenue and user count by subscription type (acquisition channel)

**Visualisations**
- Monthly churn trend (line chart)
- Churn rate by plan type and by state (bar charts)
- Correlation heatmap of plan type, contract type, churn flag, escalations and churn risk
- Pairplot of the encoded variables
- Catplot of monthly charges by plan type, split by gender and churn risk
- Pivot tables of churn rate, customers and monthly charges by plan type

**Encoding note:** categorical variables used for correlation were re-encoded according to their logical order (Basic < Standard < Premium; Monthly < Annual; low < mid < high). This avoided misleading interpretations that can result from arbitrary alphabetical category codes.

## Key Insights

All figures below come from the notebook outputs (21 customers, 6 churned).

### Headline metrics

| Metric | Value |
|---|---|
| Churn rate | 28.57% |
| Retention rate | 71.43% |
| ARPU | 18.85 |
| Average tenure | 1,550.14 days |
| Revenue lost to churn (monthly charges) | 73.94 |
| Escalation rate | 19.05% |
| Correlation: escalations vs churn | 0.77 |

### Who is leaving

| Plan type | Customers | Churn rate | Total monthly charges |
|---|---|---|---|
| Basic | 5 | **60.00%** | 52.95 |
| Standard | 9 | 22.22% | 123.91 |
| Premium | 7 | 14.29% | 218.93 |

| Subscription type | Customers | Churn rate | Total monthly charges |
|---|---|---|---|
| Organic | 9 | 0.00% | 145.91 |
| Paid | 6 | 16.67% | 174.94 |
| Referral | 6 | **83.33%** | 74.94 |

- Churn falls as the plan tier rises, and Premium brings the most revenue with the lowest churn.
- Referral customers churn far more than any other channel (5 of 6 cancelled); Organic customers did not churn.
- By state, Karnataka (100%, 2 customers) and Meghalaya (66.67%, 3 customers) have the highest churn. Uttar Pradesh has the highest revenue (115.98) and no churn. Each state has only 1 to 4 customers.

### Why they leave

Correlation with `churn_flag` (ordered encoding):

| Variable | Correlation |
|---|---|
| churn_risk | 0.95 |
| escalations | 0.77 |
| contract_type (Monthly to Annual) | -0.52 |
| plan_type (Basic to Premium) | -0.36 |

- Escalations show the strongest behavioural link with churn.
- Annual contracts are associated with lower churn than monthly contracts.
- Plan type and contract type are also correlated with each other (0.62).

### When they leave

- All cancellations fall in 2024: February (1), May (1), September (2), October (1), November (1).
- September 2024 is the peak; September to November account for 4 of the 6 cancellations.

A fuller write-up is in [`Churn_Analysis_Insights_Documentation.pdf`](./Churn_Analysis_Insights_Documentation.pdf).

## Limitations

- **Small sample:** 21 customers and 6 churned means one customer moves a percentage by several points, and state-level results rest on 1 to 4 customers. Findings are directional only.
- **Correlations are not causal:** they are computed on ordinal/binary encodings of a tiny dataset.
- **Tenure depends on the run date:** active customers use `pd.Timestamp.today()`, so the 1,550.14-day figure changes each time the notebook is run.
- **Complaint metric:** the average-complaints calculation in the notebook is not used in the reported insights because the calculation needs correction.
- **Support data:** duplicate support records/rows were removed during cleaning, reducing the support table from 9 to 7 rows before the final join. As a result, the final customer-level analysis contains one row per customer.
- **Data quirks:** the source spells the channel "Refferal", and the `state` column contains "Kathmandu", so geography is not consistently state-level.

## Project Files

```
.
├── README.md
├── Churn_Analysis.ipynb                        # Full analysis notebook
├── Churn_Analysis.html                         # Exported notebook (outputs and charts)
├── customer_churn.db                           # SQLite database (3 tables)
├── churn.csv                                   # Cleaned, joined dataset exported from the notebook
├── Churn_Analysis_Insights_Documentation.pdf   # Insights write-up
└── reference_insights_documentation.pdf        # Guided reference: churn definitions and table schema
```


## How to Run

1. Clone the repository and place `customer_churn.db` in the same folder as the notebook (the notebook connects to it by relative path).
2. Install the dependencies:
   ```bash
   pip install pandas numpy matplotlib seaborn jupyter
   ```
   `sqlite3` ships with Python.
3. Launch Jupyter and run the notebook top to bottom:
   ```bash
   jupyter notebook Churn_Analysis.ipynb
   ```

## Project Takeaways

This project strengthened practical skills in working with relational data, cleaning and joining datasets, feature engineering, aggregation, exploratory analysis and visualisation. It also highlighted the importance of checking data grain, validating transformations and interpreting small-sample results cautiously.

## Skills Practiced

- Reading relational data into pandas with `sqlite3` and `pd.read_sql`
- Data cleaning: type conversion, standardising categories, handling missing values, dropping unused columns
- Joining multiple tables and diagnosing row duplication after a merge
- Feature engineering (`churn_flag`, `tenure_days`, `churn_risk`) with `np.where` and `np.select`
- Aggregation with `groupby`, `agg` and `pivot_table`
- Ordinal encoding and correlation analysis, including correcting an encoding mistake
- Visualisation with Matplotlib and Seaborn (line, bar, heatmap, pairplot, catplot)
