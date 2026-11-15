# Vision Insurance dataset guide

38,476 claims; 47 CSV columns. Entirely synthetic. 

## Feature definitions

### Billing and size

| Column | Logical type / unit | Definition | Missing rows |
|---|---|---|---:|
| total_billed | Decimal USD | Claim billed total in USD; reconciled to the sum of billed_amount over its lines. | 0 |
| log_total_billed | Decimal | Natural logarithm ln(1 + total_billed). | 0 |
| max_line_billed | Decimal USD | Maximum billed line amount. | 0 |
| billed_per_line | Decimal USD | total_billed / line_count, USD per line row. | 0 |
| line_count | Integer count | Count of claim-line rows. | 0 |
| total_units | Integer count | Sum of units across all claim lines; mixed procedure units, not line count. | 0 |

### Claim and member context

| Column | Logical type / unit | Definition | Missing rows |
|---|---|---|---:|
| primary_dx | Categorical string | Primary submitted ICD-10 diagnosis code; categorical string. | 0 |
| secondary_dx | Categorical string | Optional secondary submitted ICD-10 diagnosis code. Blank means not supplied, not proof of no medical condition. | 26,565 |
| claim_frequency_code | Integer-coded category | 1 = original, 7 = replacement; categorical. Contrary to original documentation, 156 replacement claims are fraud-labeled. Labels were not changed. | 0 |
| plan_type | Categorical string | Synthetic plan benefits category: Exam only or Exam + materials. | 0 |
| frame_allowance | Decimal USD | Listed frame benefit allowance in USD, not the amount billed or paid. | 0 |
| age | Integer years | Member age in years from snapshot; not established as age on service date. | 0 |
| has_chronic_condition | Binary integer | Any listed chronic condition. | 0 |
| submission_lag_days | Integer-valued days | submission_date minus service_date in whole calendar days. | 0 |

### Procedures and combinations

| Column | Logical type / unit | Definition | Missing rows |
|---|---|---|---:|
| has_comprehensive_exam | Binary integer | 92004 or 92014 present. | 0 |
| has_exam | Binary integer | Any exam line. | 0 |
| s_code_with_refraction | Binary integer | 1 if S0620/S0621 and 92015 coexist on claim; otherwise 0. | 0 |
| fitting_without_contacts | Binary integer | 1 if 92310 exists without a contacts category line; otherwise 0. | 0 |
| materials_without_exam | Binary integer | 1 if frame or lens category exists without exam category on this claim; otherwise 0. | 0 |
| imaging_with_routine_primary_dx | Binary integer | 1 if imaging exists AND primary_dx starts with Z01 or H52; otherwise 0. Does not inspect secondary diagnosis. | 0 |
| imaging_line_count | Integer count | Line count for category imaging. | 0 |
| frame_line_count | Integer count | Line count for category frame. | 0 |
| lens_line_count | Integer count | Line count for category lens. | 0 |
| lens_addon_line_count | Integer count | Line count for category lens_addon. | 0 |
| contacts_line_count | Integer count | Line count for category contacts. | 0 |
| service_line_count | Integer count | Line count for category service. | 0 |

### Coverage

| Column | Logical type / unit | Definition | Missing rows |
|---|---|---|---:|
| outside_coverage | Binary integer | 1 if service is before eligibility_start or after eligibility_end; otherwise 0. Coverage boundary dates are inclusive. | 0 |
| days_after_coverage_end | Integer-valued days | max(service_date minus eligibility_end in days, 0). Uses member coverage snapshot. | 0 |

### Prior history

| Column | Logical type / unit | Definition | Missing rows |
|---|---|---|---:|
| provider_prior_claim_count | Integer count | All earlier-day provider claims. | 0 |
| provider_prior_claims_30d | Integer count | Provider claims with 0 < (current submission date minus prior submission date) <= 30 days. | 0 |
| provider_prior_mean_billed | Decimal USD | Arithmetic mean total_billed of ALL earlier-day provider claims, USD. Missing without prior claims. | 412 |
| billed_vs_provider_prior_mean | Decimal ratio | Current total_billed / provider_prior_mean_billed; unitless. Missing without valid positive historical mean. | 412 |
| provider_prior_comprehensive_share | Decimal ratio | Earlier provider claims with any 92004/92014 divided by ALL earlier provider claims, not just exams. Missing without prior claims. | 412 |
| provider_prior_imaging_share | Decimal ratio | Earlier provider claims with imaging divided by ALL earlier provider claims. Missing without prior claims. | 412 |
| member_prior_claim_count | Integer count | All earlier-day member claims. | 0 |
| member_prior_claims_30d | Integer count | Member claims with 0 < (current submission date minus prior submission date) <= 30 days. | 0 |
| member_prior_distinct_providers | Integer count | Distinct providers on earlier member claims. | 0 |
| member_provider_prior_claim_count | Integer count | Earlier member-provider pair claims. | 0 |
| days_since_prior_routine_exam | Integer-valued days | Current service date minus latest qualifying routine-exam service date among earlier-day member submissions with service_date no later than current service. Missing without prior qualifying exam. | 23,715 |
| prior_exact_duplicate_within_3_service_days | Binary integer | 1 if an earlier-day submission for the same member/provider has an identical sorted multiset of code, units and billed cents, with current service_date minus prior service_date in 0–3 days inclusive. Does not match same-day submissions. | 0 |

## Metadata and target

| Column | Logical type / unit | Definition | Missing rows |
|---|---|---|---:|
| claim_id | String ID | Unique claim identifier; join key only. One row per claim. | 0 |
| member_id | String ID | Synthetic member identifier; history, grouping and traceability only. | 0 |
| provider_id | String ID | Synthetic provider identifier; history, provider-group splitting and traceability only. | 0 |
| service_date | Date YYYY-MM-DD | Calendar date services were reported as performed. | 0 |
| submission_date | Date YYYY-MM-DD | Calendar date submitted; basis for chronological splits and historical information availability. | 0 |
| fraud_type | | | |
| is_fraud | Binary integer | Synthetic target: 1 = fraud-labeled, 0 = legitimate-labeled. Not independently investigated real-world fraud. | 0 |
