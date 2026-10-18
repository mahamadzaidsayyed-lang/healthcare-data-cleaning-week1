# Week 1 Task: Data Cleaning and Preparation (Healthcare)

Data cleaning exercise on a **simulated** hospital admissions dataset (no real patient data).

## Files
| File | Purpose |
|---|---|
| `Data_Cleaning_Report.docx` | Main deliverable: detailed cleaning report |
| `generate_data.py` | Creates the raw dataset with planted quality issues |
| `data/raw_patients.csv` | Raw data (65 rows) |
| `clean_data.py` | Cleaning pipeline (pandas) |
| `data/clean_patients.csv` | Cleaned data (60 rows) |
| `cleaning_log.json` | Audit log of counts at each step |

## Run
```
pip install pandas numpy
python generate_data.py
python clean_data.py
```

## Summary
Issues handled: missing values, duplicates (exact and near), outliers, inconsistent categories, mixed date formats, and illogical dates. Imputed and corrected values are flagged in separate columns for traceability.
