# Vision Insurance Fraud Detection

Predict whether a vision insurance claim is fraud-labeled using information available when the claim is submitted. The prediction unit is one claim. The target is `is_fraud` (`1` = fraud-labeled, `0` = legitimate-labeled). The intended use is to prioritize claims for human review.

## Repository layout

| Path | Description |
| --- | --- |
| [dataset/VisionInsurance_claims_dataset.csv](dataset/VisionInsurance_claims_dataset.csv) | Claim-level table: 38,476 rows, 47 columns |
| [dataset/Dataset_Guide.md](dataset/Dataset_Guide.md) | Column definitions, units, and missingness |
| [notebook/Data_Understanding_and_exploratory data analysis_EDA.ipynb](notebook/Data_Understanding_and_exploratory%20data%20analysis_EDA.ipynb) | Data understanding and exploratory analysis |

## Dataset

Source: [https://www.kaggle.com/datasets/ashok0282/synthetic-vision-insurance-claims-fraud-detection](https://www.kaggle.com/datasets/ashok0282/synthetic-vision-insurance-claims-fraud-detection)

Each row is one claim. Columns cover identifiers, service and submission dates, billed amounts, diagnoses and plan context, procedure combination flags, coverage timing, member and provider history, a `fraud_type` label, and the binary target `is_fraud`.

- `claim_id`, `member_id`, `provider_id` — identifiers for grouping and traceability only
- `service_date`, `submission_date` — keep for chronological splits and historical information, not as raw model features
- `fraud_type` — describes the fraud pattern and leaks the label; do not use it as a predictor

Full feature definitions are in [dataset/Dataset_Guide.md](dataset/Dataset_Guide.md).

## Current findings (EDA)

- **Quality:** No duplicate rows, no duplicate `claim_id`s, no missing targets, no missing submission dates, and no submissions dated before the service date.
- **Missing values:** `secondary_dx` is missing on about 69% of rows (not supplied). `days_since_prior_routine_exam` is missing on about 62% (no qualifying prior exam). Provider history fields (`provider_prior_mean_billed`, `billed_vs_provider_prior_mean`, `provider_prior_comprehensive_share`, `provider_prior_imaging_share`) are missing on about 1.07% of rows (no prior claims).
- **Class imbalance:** 37,322 claims (97%) are non-fraud and 1,154 (3%) are fraud-labeled. A model that always predicts non-fraud would reach about 97% accuracy and detect no fraud. Prefer precision, recall, and average precision over accuracy.

### Feature associations

- **High fraud rate on uncommon flags:** Relative to the 3% base rate, fraud is much more common when `prior_exact_duplicate_within_3_service_days` is 1 (~95%), `fitting_without_contacts` is 1 (~76%), `claim_frequency_code` is 7 / replacement (~41%), `materials_without_exam` is 1 (~40%), `outside_coverage` is 1 (~36%), there is no exam line (~32%), or `imaging_with_routine_primary_dx` is 1 (~28%). 
- **Billed amount is higher on fraud-labeled claims:** Mean `total_billed` is about $596 vs $341 for non-fraud; `max_line_billed` and `billed_per_line` move in the same direction. `age` is not associated with the label. `plan_type` fraud rates are essentially the same.
- **Collinear billing and history columns:**
  - `total_billed` and `log_total_billed` correlate at about 0.91; 
  - `line_count` and `total_units` at about 0.90; 
  - billed size also tracks `max_line_billed` and `billed_per_line`. `has_exam` and `materials_without_exam` are near-opposites. 
  - Member history counts (`member_prior_claim_count` vs `member_provider_prior_claim_count`) overlap strongly.

## How to run

Python 3 with pandas and matplotlib is enough for the current notebook.

1. Clone the repository.
2. Open `notebook/Data_Understanding_and_exploratory data analysis_EDA.ipynb` in Jupyter or another notebook environment.
3. Point `DATA_PATH` at the CSV in this repo. The notebook currently loads VisionInsurance_claims_dataset`.csv`. 

```python
from pathlib import Path

DATA_PATH = Path("../dataset/VisionInsurance_claims_dataset.csv")
```

Run the notebook from the `notebook/` directory so that relative path works, or use an absolute path to `dataset/VisionInsurance_claims_dataset.csv`.
