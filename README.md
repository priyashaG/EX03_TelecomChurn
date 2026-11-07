# Airtel Customer Churn Diagnostics and Retention Strategy

An analysis of 4,250 labelled Airtel customer accounts to find what separates customers who churn from those who stay, and where retention effort should go. Every figure here is computed from `data/train.csv`; the Excel workbook uses live formulas on that data, so each number can be traced.

---

## Executive Summary

- **Customers analysed:** 4,250 (train.csv)
- **Churned customers:** 598
- **Overall churn rate:** 14.1%
- **Strongest drivers:** 4+ customer service calls (50.7% churn), day usage above 250 minutes (48.9%) and an international plan (42.2%). Customers without these flags churn at about 10% to 11%.
- **Weak or no drivers:** area code (13.6% to 15.1%) and account length (correlation 0.02).
- **Estimated revenue lost:** about 470K per year (annualised billing of churned customers). This is an *assumption-based* estimate; see Limitations.

---

## Dataset and Scope

- `train.csv`: 4,250 rows, 20 columns, labelled with `churn` (yes/no).
- `test.csv` (750 rows) has no churn labels, so it is not used for analysis.
- Fields: state, area code, account length, international plan, voice mail plan, voice mail messages, day / evening / night / international minutes, calls and charges, customer service calls, churn.
- **Not in the data:** contract type, payment method, internet technology (fiber/DSL), tech support, monthly bill, tenure cohorts, dates, call reasons. Nothing in this repository claims results on those.

---

## Answers to the Case-Study Questions

### Basic

| Question | Answer |
| :--- | :--- |
| Q1 Churn count and share | 598 churned (14.1%), 3,652 stayed (85.9%) |
| Q2 Account length | Churned 102.1 months vs 99.9 retained; correlation 0.019, so tenure has almost no effect |
| Q3 International plan | 42.2% churn with the plan (n=396) vs 11.2% without (3.8x) |
| Q4 Service calls | Churned customers average 2.28 calls vs 1.44 for retained |
| Q5 Day / evening / night charges | Day 35.53 vs 29.84 (+19%); evening 17.85 vs 16.88 (+6%); night 9.29 vs 8.98 (+3%) |
| Q6 Day minutes | Churned 209.0 vs 175.6 average minutes; correlation 0.215 |
| Q7 Voice mail plan | 7.4% churn with the plan vs 16.4% without (0.45x) |
| Q8 International minutes and charges | Minutes 10.6 vs 10.2, charge 2.87 vs 2.75: small difference (correlation 0.055) |
| Q9 Day calls | Correlation 0.012: no relationship |
| Q10 Evening charge | 17.85 vs 16.88 average; the highest evening-charge quartile churns at 17.9% vs 10.4% for the lowest |

### Medium

| Question | Answer |
| :--- | :--- |
| Area code | 408: 14.0%, 415: 13.6%, 510: 15.1%. No area stands out |
| Service-call threshold | 0 to 3 calls: about 11%. **4 calls: 44.0%, 5 calls: 60.5%, 6+: 64.4%.** The jump happens at the fourth call (4+ calls: 50.7%, 4.6x) |
| Account length quartiles | 12.8%, 14.3%, 15.9%, 13.3%: no pattern; newer customers are not more likely to churn |
| International charge (plan holders) | Above-median charge 50.8% vs 33.7% at or below |
| Evening usage | Churn rises with evening charge (10.4% to 17.9% across quartiles); evening calls are unrelated (r = -0.007) |
| Usage clusters (k-means, k=4) | Heavy day & evening users churn at 22.2% (n=1,098); the other three clusters at 10.8% to 11.9% |
| Voice mail x messages | No plan 16.4%; plan holders 2.3% to 9.1% by message volume |
| Day minutes inflection | Up to 250 minutes: 8.0% to 12.1%. **251 to 300: 43.8%; above 300: 81.0%** |
| International calls x plan | Plan holders with 0 to 3 calls churn at 58.8%; non-holders are about 11% at any call volume |
| Call reasons | **Not possible:** the data holds only the number of calls, not their reason |

### Advanced

| Question | Answer |
| :--- | :--- |
| Logistic regression | 10 predictors; cross-validated AUC 0.826 (in-sample 0.830). Largest effects: international plan (odds x7.9), voice mail plan (x0.09), each service call (x1.70). Account length is not significant (p = 0.39). At a 30% threshold: accuracy 85.4%, precision 48.0%, recall 44.5% |
| Time-series analysis | **Not possible:** no date or billing-cycle field in the data |
| Clustering | See the cluster result above (Advanced sheet has centroids) |
| Interaction effects | International plan x voice mail: 45.5% (plan, no voice mail) vs 4.8% (no plan, voice mail). Extra international minutes raise churn only for plan holders (interaction p < 0.001). the two triggers do not stack: 4+ calls alone 50.8%, 250+ day minutes alone 48.9%, both together 50.0% (n=30) |
| Economic impact | Annualised billing of churned customers about 470K; 24-period customer value forfeited about 941K. Bringing 4+ call customers to the baseline churn rate would protect about 105K a year |

Predicted risk bands match actual churn well: under 10% predicted churns at 3.5%, 50% and above at 57.5%.

---

## High-Risk Segments

| Segment | Customers | Churn | Baseline | Relative risk |
| :--- | :--- | :--- | :--- | :--- |
| 4+ customer service calls | 335 | 50.7% | 10.9% (0 to 3 calls) | 4.6x |
| Day minutes above 250 | 423 | 48.9% | 10.2% (250 or fewer) | 4.8x |
| International plan, no voice mail | 292 | 45.5% | 11.7% (all others) | 3.9x |
| International plan | 396 | 42.2% | 11.2% (no plan) | 3.8x |
| Heavy day & evening cluster | 1,098 | 22.2% | 11.2% (all others) | 2.0x |

Segments overlap, so rates are not additive.

---

## Retention Roadmap

- **Q1: Service calls.** Flag customers at their 3rd service call; trigger proactive resolution and a save offer.
- **Q2: Heavy users.** Offer higher-allowance day plans to customers approaching 250 day minutes.
- **Q3: International plan.** Re-price or bundle the international plan with voice mail.
- **Q4: Risk scoring.** Score customers with the logistic model and review those at 30% or higher predicted risk.

---

## Repository Structure

```text
├── data/
│   ├── train.csv
│   └── test.csv
├── working_files/
│   └── Airtel_Customer_Churn_Working_File.xlsx
├── screenshots/
│   └── airtel_churn_dashboard_clean.png
├── presentation/
│   └── Airtel Customer Churn Analytics.pptx
└── README.md
```

### Excel workbook

| Sheet | Contents |
| :--- | :--- |
| Executive_Summary | KPIs and core drivers (live formulas) |
| Dashboard | KPI tiles and four charts |
| Basic_Questions | Q1 to Q10 |
| Medium_Questions | Area code, service-call thresholds, quartiles, clusters, day-minute bands and more |
| Advanced_Questions | Logistic model, confusion matrix, calibration, interactions, economic impact |
| Notes_Definitions | Definitions, assumptions, limitations |
| Data | The 4,250 labelled rows plus helper columns (churn flag, total charge, cluster, predicted probability) |

Logistic-regression coefficients and the k-means cluster labels were fitted outside Excel (Python), because Excel has no logistic-fit or clustering function. Everything built on them, including predicted probabilities, the confusion matrix and the segment tables, is a live Excel formula.

---

## Limitations

- **Association, not causation.** These results show which customers churn more, not why.
- **Revenue figures are assumptions.** The data does not state the billing period or currency; the estimate assumes monthly billing and a 24-period customer horizon. Both are editable cells in the workbook.
- **Small groups are noisy.** For example, 6+ service calls (n=45) and international-plan sub-bands (n about 100 to 200).
- **No separate test set.** test.csv has no labels, so model performance is reported in-sample and by 5-fold cross-validation.
- **Questions the data cannot answer:** call reasons, time trends and seasonality, and any contract, payment or technology analysis.
