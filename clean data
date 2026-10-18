"""Data cleaning pipeline for the simulated hospital admissions dataset.

Run:  python clean_data.py
Input:  data/raw_patients.csv
Output: data/clean_patients.csv, cleaning_log.json
"""
import json
import numpy as np
import pandas as pd

log = {}
df = pd.read_csv("data/raw_patients.csv")

# ------------------------------------------------------------------ 1. Profile
log["rows_raw"] = int(len(df))
log["missing_raw"] = {k: int(v) for k, v in df.isna().sum().items() if v > 0}
log["exact_duplicates_raw"] = int(df.duplicated().sum())
print("RAW SHAPE:", df.shape)
print(df.isna().sum())

# ------------------------------------------------- 2. Standardize text values
for col in ["Gender", "Diagnosis", "Department"]:
    df[col] = df[col].astype("string").str.strip()

gender_map = {"m": "Male", "male": "Male", "f": "Female", "female": "Female"}
df["Gender"] = df["Gender"].str.lower().map(gender_map).where(df["Gender"].notna())

diag_map = {"htn": "Hypertension", "hypertension": "Hypertension", "diabetes": "Diabetes",
            "pneumonia": "Pneumonia", "asthma": "Asthma", "fracture": "Fracture",
            "heart failure": "Heart Failure"}
df["Diagnosis"] = df["Diagnosis"].str.lower().map(diag_map).where(df["Diagnosis"].notna())
log["gender_unmapped"] = int(df["Gender"].isna().sum())

# ------------------------------------------------------------- 3. Parse dates
for col in ["AdmissionDate", "DischargeDate"]:
    df[col] = pd.to_datetime(df[col], format="mixed", errors="coerce")

# -------------------------------------- 4. Remove duplicates (after cleaning)
before = len(df)
df = df.drop_duplicates().reset_index(drop=True)
log["duplicates_removed"] = int(before - len(df))
dup_ids = df["PatientID"].duplicated().sum()
log["duplicate_patient_ids_remaining"] = int(dup_ids)

# --------------------------------- 5. Outliers: clinical plausibility ranges
ranges = {"Age": (0, 110), "SystolicBP": (60, 250), "DiastolicBP": (30, 150)}
log["outliers_flagged"] = {}
for col, (lo, hi) in ranges.items():
    mask = df[col].notna() & ~df[col].between(lo, hi)
    log["outliers_flagged"][col] = int(mask.sum())
    df[f"{col}_OutlierFlag"] = mask
    df.loc[mask, col] = np.nan  # treat impossible values as missing

# ------------------------------------- 6. Logical check: dates out of order
bad_dates = df["DischargeDate"] < df["AdmissionDate"]
log["discharge_before_admission"] = int(bad_dates.sum())
df["DateOrderFlag"] = bad_dates
df.loc[bad_dates, "DischargeDate"] = pd.NaT

# ------------------------------------------------------------ 7. Missing data
for col in ["Age", "SystolicBP", "DiastolicBP"]:
    df[f"{col}_Imputed"] = df[col].isna()
    median = df[col].median()
    log[f"{col}_median_used"] = float(median)
    df[col] = df[col].fillna(median).round().astype(int)

df["Gender"] = df["Gender"].fillna("Unknown")
df["Diagnosis"] = df["Diagnosis"].fillna("Unknown")
# DischargeDate is deliberately NOT imputed: a missing date may mean the patient is still admitted.
df["LengthOfStay"] = (df["DischargeDate"] - df["AdmissionDate"]).dt.days

# ---------------------------------------------------------------- 8. Validate
assert df["PatientID"].is_unique
assert df["Age"].between(0, 110).all()
assert df["SystolicBP"].between(60, 250).all()
assert set(df["Gender"]) <= {"Male", "Female", "Unknown"}
log["rows_clean"] = int(len(df))
log["missing_clean"] = {k: int(v) for k, v in df.isna().sum().items() if v > 0}

df.to_csv("data/clean_patients.csv", index=False)
with open("cleaning_log.json", "w") as f:
    json.dump(log, f, indent=2)
print(json.dumps(log, indent=2))
