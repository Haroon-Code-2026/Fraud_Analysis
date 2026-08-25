
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

  ## Key Findings

| H1 | Prior fraudulent activity predicts future fraud | Not supported | 0.8851 |
| H2 | Authentication method is associated with fraud rate | Not supported | 0.4354 |
| H3 | Flagged IP addresses carry higher fraud risk | Not supported | 0.5124 |
| H4 | Fraud occurs further from the user's usual location | Not supported | 0.5103 |
| H5 | Recent failed transactions predict fraud | Supported | <0.001 (r = 0.510) |

* Fraud makes up 32.13% of this dataset - a moderate imbalance, not the severe imbalance often assumed in fraud detection write-ups.
* Only one of five tested hypotheses held up: accounts with more failed transactions in the last 7 days are substantially more likely to commit fraud.
* Logistic Regression reached 0.56 precision / 0.72 recall on the fraud class, ROC-AUC 0.802. Random Forest reached 1.00 precision / 0.62 recall, ROC-AUC 0.812 - a genuine trade-off between catching more fraud and avoiding false alarms, not a clear winner either way.
* The Random Forest's own feature importance strongly corroborated H5: `Failed_Transaction_Count_7d` accounted for 92.6% of its total importance, with every other feature contributing 1% or less - a statistical test and a trained model, using different methods, independently reached the same conclusion.


  ## Project Plan

* High-level steps: explore raw data → ETL → hypothesis testing & visualisation → modelling.
* **Data management:** raw CSV kept untouched in `data/raw/`; cleaning writes to a separate `data/cleaned/` file, so the source is always re-checkable.
* **Why these methods:** statistical tests (not just charts) because a chart alone can't tell you if a difference is real or noise. Logistic Regression + Random Forest as a *deliberate pair* — one interpretable, one able to catch non-linear interactions — not two similar models.

## The rationale to map the business requirements to the Data Visualisations

* BR1 & BR2 (identify significant features / test commonly assumed risk factors): the five hypothesis tests - each one confirms or rules out a specific "commonly assumed" risk factor, backed by a chart and a statistical test rather than visual impression alone.
* BR3 (build and compare models): Logistic Regression vs Random Forest, evaluated on precision/recall/ROC-AUC, not accuracy.
* BR4 (present findings, check model agreement): charts tied to each hypothesis + a feature importance chart compared directly against the statistical findings.


## Analysis techniques used

* Cleaning/quality: missing values, duplicates, IQR outlier checks across all numeric columns.
* Stats: chi-square (H1, H2, H3 - categorical comparisons), Welch's t-test (H4 - continuous vs group), point-biserial correlation (H5 - continuous vs binary).
* Viz: bar charts, box plot, line chart, correlation heatmap, feature importance chart.
* ML: Logistic Regression + Random Forest, evaluated on precision/recall/F1/ROC-AUC.
* **Did the data limit you?** Yes - only 5 `Location` categories limited geographic analysis; fraud rate (32.13%) is much higher than real-world fraud, so treated as moderate imbalance, not severe.
* **Generative AI use:** used to help structure the project, sanity-check which stats test fits which comparison, and debug code. Checked against real data output throughout, not trusted blind - see Unfixed Bugs.

## Ethical considerations

* Dataset is synthetic - no real personal data.
* Four of five hypotheses were NOT supported - reported honestly rather than hidden.
* Bias: since assumed risk factors (auth method, IP flag, location, prior history) didn't hold up statistically, a real system shouldn't build fraud rules on them just because they're commonly cited elsewhere.
* Model fragility: over 90% of feature importance sits on one feature (`Failed_Transaction_Count_7d`) - flagged as a weakness, not a strength, since a real deployment would be fragile if that signal became unavailable.

## Dashboard Design

* Built in Tableau Public, connected directly to `data/cleaned/cleaned_transactions.csv` and a second export of the model's feature importance results.
* One main dashboard plus four mini-dashboards: Overview (class balance + hypothesis summary table), Hypotheses Not Supported (H1-H4), The One Signal That Held Up (H5 + fraud by hour), and Model Corroboration (feature importance).
* Interactive filtering via a shared field applied across worksheets, with navigation buttons linking each mini-dashboard back to the main one.


## Unfixed Bugs

* No known unfixed bugs - all three notebooks run end-to-end with no errors.
* One warning encountered and fixed: `set_xticklabels()` without `set_xticks()` first raised a `UserWarning` in the H4 boxplot - fixed by setting tick positions explicitly before labelling them.
* One environment error encountered and fixed: `savefig()` failed with `FileNotFoundError` because `outputs/figures/` didn't exist yet - fixed by adding `os.makedirs('outputs/figures', exist_ok=True)` before any chart-saving code runs.
* Gap I had to catch and fix during development: I initially built the ETL notebook against a placeholder dataset with artificially injected missing values/duplicates, before I had the real file. Once I loaded the real data, none of those issues existed, so I had to rewrite the affected markdown.


## Development Roadmap

* Main challenge: resisting the pull to make every hypothesis "work" once 4/5 came back not significant.
* Next steps: finish publishing the Tableau dashboard, investigate whether combining the four "not supported" features together reveals an interaction effect the univariate tests missed, try a gradient boosting model for a fuller comparison.

## Deployment


## Main Data Analysis Libraries

* pandas - ETL, EDA, feature checks
* numpy - numeric operations
* matplotlib / seaborn - charts and statistics visualisations
* scipy - chi-square tests, t-test, point-biserial correlation
* scikit-learn - Logistic Regression, Random Forest, train/test split, evaluation metrics
* joblib - saving the trained model

## Credits

* Dataset: samayashar (Kaggle) - Fraud Detection Transactions Dataset.
* Code Institute for the project structure and assessment criteria.
* Claude AI for structure help, debugging, and statistical reasoning checks.
