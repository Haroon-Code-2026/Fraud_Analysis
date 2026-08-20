
## Dataset Content

* The dataset used is the **Fraud Detection Transactions Dataset** from Kaggle: https://www.kaggle.com/datasets/samayashar/fraud-detection-transactions-dataset
* It is the real, downloaded CSV file - no rows or columns have been added to or removed from the original dataset.
* It contains 50,000 transaction records across 21 columns: transaction details (amount, type, timestamp, device, location, merchant category), behavioural/account signals (previous fraudulent activity, daily transaction count, failed transactions in the last 7 days, 7-day average spend), card details (type, age), a pre-calculated risk score, and the target label (`Fraud_Label`).
* The file is 7.02 MB, well within the repository size limit.
* The dataset is documented as synthetic, so there is no real personal data and no privacy concern despite the realistic-looking records.

## Business Requirements

* Identify which transaction and behavioural features are significantly associated with fraud.
* Determine whether commonly assumed risk factors - prior fraud history, authentication method, IP address flags, and transaction distance - actually influence fraud rate, or whether fraud risk is instead concentrated in a small number of stronger behavioural signals.
* Build and compare classification models to predict fraud from the available features.
* Present findings through clear visualisations, and check whether the model's own reasoning (feature importance) agrees with the statistical hypothesis testing.

## Hypothesis and how to validate?

* H1: Accounts with prior fraudulent activity are more likely to commit fraud again.
  * Validation: chi-square test + bar chart.
* H2: Fraud rate differs by authentication method.
  * Validation: chi-square test + bar chart.
* H3: Transactions from a flagged IP address have a higher fraud rate.
  * Validation: chi-square test + bar chart.
* H4: Fraud happens further from the user's usual location.
  * Validation: box plot + Welch's t-test.
* H5: Accounts with more recent failed transactions are more likely to commit fraud.
  * Validation: point-biserial correlation + bar chart.

  ## Project Plan

* High-level steps: explore raw data → ETL → hypothesis testing & visualisation → modelling.
* **Data management:** raw CSV kept untouched in `data/raw/`; cleaning writes to a separate `data/cleaned/` file, so the source is always re-checkable.
* **Why these methods:** statistical tests (not just charts) because a chart alone can't tell you if a difference is real or noise. Logistic Regression + Random Forest as a *deliberate pair* — one interpretable, one able to catch non-linear interactions — not two similar models.

## The rationale to map the business requirements to the Data Visualisations





## Analysis techniques used



## Ethical considerations



## Dashboard Design



## Unfixed Bugs



## Development Roadmap



## Deployment

## Heroku

## Main Data Analysis Libraries

## Credits

