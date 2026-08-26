
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

  ## Project Plan

* High-level steps: explore raw data → ETL → hypothesis testing & visualisation → modelling.
* **Data management:** raw CSV kept untouched in `data/raw/`; cleaning writes to a separate `data/cleaned/` file, so the source is always re-checkable.
* **Why these methods:** statistical tests (not just charts) because a chart alone can't tell you if a difference is real or noise. Logistic Regression + Random Forest as a *deliberate pair* — one interpretable, one able to catch non-linear interactions — not two similar models.


* [Kanban Board](https://github.com/users/Haroon-Code-2026/projects/2)

* [Tableau Dashboard](https://public.tableau.com/views/FraudAnalysisDashboard_17877394547560/DashboardSummary?:language=en-GB&:sid=&:redirect=auth&:display_count=n&:origin=viz_share_link)



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

  ## Model

Two supervised classification models were built to predict `Fraud_Label`: Logistic Regression and
Random Forest. These were chosen as a deliberate pair rather than two similar models - Logistic
Regression is interpretable, giving each feature a clear coefficient showing its direction and
rough strength of association with fraud, which matters if this ever needs explaining to a
non-technical stakeholder. Random Forest can pick up on interactions between features that
Logistic Regression can't (for example, a specific combination of device type and merchant
category), at the cost of being harder to explain the reasoning behind any individual prediction.

`Risk_Score` was deliberately excluded from the feature set. The ETL notebook found it correlates
with `Fraud_Label` at r = 0.386, and since it is itself a derived fraud-risk estimate, including it
would let the model partly rely on an existing score rather than learning from the raw transaction
data - the result would look artificially stronger without the model having genuinely learned
more.

Both models used `class_weight='balanced'`, since the dataset is moderately imbalanced (32.13%
fraud), so neither model would default toward simply predicting the majority class. A stratified
train/test split (75/25) preserved this same fraud rate in both sets.

**Comparison:**

| Model | Precision (Fraud) | Recall (Fraud) | ROC-AUC |
|---|---|---|---|
| Logistic Regression | 0.56 | 0.72 | 0.802 |
| Random Forest | 1.00 | 0.62 | 0.812 |

Random Forest reached the higher ROC-AUC and notably perfect precision on the fraud class - every
transaction it flagged as fraud in the test set actually was fraud. Its recall was lower, meaning
it missed a larger share of actual fraud cases. Logistic Regression traded this the other way:
lower precision but higher recall, catching more fraud overall at the cost of more false alarms.
Which model is preferable depends on the operational cost of a missed fraud case versus a false
alarm - the data does not settle this on its own, it is a business decision.

The Random Forest's feature importance was also compared against the EDA notebook's hypothesis
testing results. `Failed_Transaction_Count_7d` accounted for 92.6% of its total importance, with
every other feature contributing 1% or less - independently confirming H5, the only hypothesis
that was statistically significant. A statistical test and a trained model, using entirely
different methods, reaching the same conclusion is stronger evidence than either result alone.

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

## Limitations

* Four of the five hypotheses tested were not statistically supported, which limited how much of a
  multi-factor story could be built from this dataset - the analysis is ultimately built around one
  strong signal rather than several converging ones.
* The model's performance is very heavily dependent on a single feature
  (`Failed_Transaction_Count_7d`, 92.6% of feature importance). This makes it fragile in a
  real-world sense - if that signal became unavailable, delayed, or unreliable, the model would
  have little else to fall back on.
* The dataset's `Location` column only has 5 categories, which limited how much genuine geographic
  analysis was possible.
* Fraud makes up 32.13% of this dataset, far higher than a real-world fraud rate would typically
  be. Conclusions about class imbalance and model behaviour here may not transfer directly to a
  dataset with a more realistic (much lower) fraud rate.
* The dataset is documented as synthetic. The relationships found - and, just as importantly, the
  relationships not found - reflect how the data was generated, not necessarily genuine real-world
  fraud patterns. This project's conclusions should be read as specific to this dataset, not
  generalised as fraud-detection findings in the abstract.
* Each hypothesis was tested individually rather than jointly. A logistic regression including all
  five variables at once could reveal an interaction effect between the four unsupported features
  that the univariate tests used here would not detect.


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

## AI Assistance


Claude AI was used throughout this project, primarily for: structuring the ETL, EDA, and Machine
Learning notebooks into clear sections; checking which statistical test was appropriate for which
type of comparison (e.g. chi-square for two categorical variables, a t-test for a numeric variable
across two groups, a point-biserial correlation for a numeric variable against a binary one); and
debugging specific errors as they came up, including a `FileNotFoundError` from `savefig()` being
called before the output folder existed, and a `UserWarning` from calling `set_xticklabels()`
without first setting fixed tick positions.

AI assistance was not taken on trust. 

AI assistance was also used to plan the Tableau Public dashboard structure (which worksheets to
build, which fields to place on which shelf) and to draft the wording used in the dashboard's text
boxes - but the dashboard itself, and the decision of what it should show, was built and reviewed
directly rather than generated end-to-end.