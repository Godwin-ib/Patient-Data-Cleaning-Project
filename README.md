
# 🏥 Data Cleaning Documentation — Patient Dataset (SQL Edition)

**Tool Used:** MySQL  
**Dataset:** `messy patient data`  
**Author:** GODWIN IBRAHIM JOHN 
**Date:** 3th April 2026  

## 1. Project Overview
This document records the complete data cleaning process performed on a healthcare dataset using **MySQL**. The raw data contained structural errors (corrupted column names), inconsistent text-based numbers, mixed-case categorical data, and invalid placeholders. 

The goal was to transform the data into a clean, analysis-ready format using **CASE statements** and **String functions**.

## 2. Dataset Description
- **Table Name:** `messy patient data`
- **Total Rows:** 101
- **Key Identifier:** `pid` (Renamed to `ppid`)

## 3. Data Quality Audit — Before Cleaning
A preliminary `SELECT *` was performed to identify quality issues:
- **Structural:** The primary ID column was named `pid` instead of the preferred `ppid`.
- **Inconsistency:** Age contained words ("forty") and invalid strings ("unknown", "-5").
- **Date Corruption:** Dates contained trailing text suffixes ("abc").
- **Redundancy:** Diagnosis had multiple synonyms for Hypertension (HBP, H.B.P, etc.).

## 4. Cleaning Steps

### Step 1 — Schema Setup & Safety
Before performing updates, the column header was corrected, and safe update mode was disabled to allow mass changes without a `WHERE` clause.

**SQL Command:**
```sql
ALTER TABLE project.`messy patient data`
RENAME COLUMN pid TO ppid;

SET SQL_SAFE_UPDATES = 0;
```

### Step 2 — Age Column
**Issue:** Mixed numeric values, written words, and invalid placeholders.
**Action:** Used `UPPER()` and `REPLACE()` within a `CASE` statement to standardize to numeric strings.

**SQL Command:**
```sql
UPDATE project.`messy patient data`
SET Age = CASE 
    WHEN Age LIKE '%years%' THEN TRIM(REPLACE(Age, 'years', ''))
    WHEN UPPER(Age) IN ('TWENTY-TWO', 'TWENTY TWO') THEN '22'
    WHEN UPPER(Age) = 'TWENTY FIVE' THEN '25'
    WHEN UPPER(Age) = 'FORTY' THEN '40'
    WHEN Age IN ('0', '-5', 'unknown', 'NaN', 'N/A') THEN NULL
    ELSE Age 
END;
```

### Step 3 — Gender Column
**Issue:** Mixed casing and abbreviations.
**Action:** Used `LIKE` patterns to group all female and male variations.

**SQL Command:**
```sql
UPDATE project.`messy patient data`
SET Gender = CASE 
    WHEN UPPER(Gender) LIKE 'F%' THEN 'Female'
    WHEN UPPER(Gender) LIKE 'M%' THEN 'Male'
    ELSE NULL 
END;
```

### Step 4 — Check-in Date Column
**Issue:** Trailing "abc" text and placeholder strings.
**Action:** Stripped the "abc" suffix and converted placeholders to `NULL`.

**SQL Command:**
```sql
UPDATE project.`messy patient data`
SET Check_in_Date = CASE 
    WHEN Check_in_Date LIKE '%abc' THEN REPLACE(Check_in_Date, 'abc', '')
    WHEN Check_in_Date IN ('Z04', 'Unknown', 'N/A') THEN NULL
    ELSE Check_in_Date
END;
```

### Step 5 — Diagnosis Column
**Issue:** Multiple labels for the same condition and numeric junk data.
**Action:** Consolidated all synonyms to "Hypertension."

**SQL Command:**
```sql
UPDATE project.`messy patient data`
SET Diagnosis = CASE 
    WHEN UPPER(Diagnosis) IN ('HBP', 'H.B.P', 'ELEVATED PRESSURE', 'HIGH BLOOD PRESSURE') THEN 'Hypertension'
    WHEN Diagnosis IN ('??', '1234', 'None', 'n/a') THEN NULL
    ELSE Diagnosis 
END;
```

### Step 6 — Heart Rate Column
**Issue:** Words ("One hundred") and vague descriptors.
**Action:** Converted words to numbers and removed non-numeric text.

**SQL Command:**
```sql
UPDATE project.`messy patient data`
SET Heart_Rate = CASE 
    WHEN Heart_Rate = 'One hundred' THEN '100'
    WHEN Heart_Rate IN ('high', 'normal', '-', 'N/A', 'NaN', '0') THEN NULL
    ELSE Heart_Rate 
END;
```

## 5. Summary of All Changes
1. **Renamed Column:** Changed `pid` to `ppid`.
2. **Age Sanitization:** Removed "years" suffix and converted "forty", "twenty-two", and "twenty five" to integers.
3. **Gender Normalization:** Unified all entries into "Male" or "Female".
4. **Diagnosis Consolidation:** Grouped 4 different HBP synonyms into "Hypertension".
5. **Junk Removal:** All "Z04", "??", "1234", and "-" values converted to `NULL` for accurate analysis.

## 6. Final Verification
```sql
SELECT ppid, Age, Gender, Check_in_Date, Diagnosis, Heart_Rate 
FROM project.`messy patient data` 
LIMIT 10;
```


---
**Note for GitHub:** *I removed the redundant text "student_mental_health_burnout_1m" from your Age query in the documentation above to ensure the code snippet remains valid for your portfolio.*
