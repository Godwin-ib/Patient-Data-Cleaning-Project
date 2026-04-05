# 🏥 Data Cleaning Documentation — Patient Dataset (SQL Edition)

**Tool Used:** MySQL  
**Dataset:** `messy patient data` 
**Author:** GODWIN IBRAHIM JOHN  
**Date:** 3rd April 2026  

---

## **Table of Contents**

1.  **[1. Project Overview](#1-project-overview)**
2.  **[2. Dataset Description](#2-dataset-description)**
3.  **[3. Identified Data Problems](#3-identified-data-problems)**
4.  **[4. Data Cleaning Steps (SQL Workflow)](#4-data-cleaning-steps)**
5.  **[5. Summary of Changes](#5-summary-of-changes)**
6.  **[6. Final Verification](#6-final-verification)**

---

## 1. Project Overview
This documentation details the end-to-end cleaning process for the `messy patient data` table. The primary goal was to resolve structural inconsistencies, standardize text-based numerical entries, and handle corrupted date and clinical records using **MySQL**. By the end of this process, the dataset was transformed from a raw, unreliable state into a structured format ready for healthcare analytics.

---

## 2. Dataset Description
* **Table Name:** `messy patient data`
* **Initial Record Count:** 999 (including empty trailing rows)
* **Effective Record Count:** 101 valid patient entries
* **Key Identifier:** `pid` (renamed to `ppid`)

---


## 3. Identified Data Problems
* **Corrupted Suffixes:** Dates contained non-date characters (e.g., `2023-11-09abc`).
* **Synonym Inflation:** High Blood Pressure was recorded in 5+ different ways (HBP, H.B.P, etc.).
* **Non-Standard Numbers:** Ages and heart rates were written in words (e.g., "Twenty five") rather than digits.
* **Schema Error:** The primary ID column was named `pid`, which required standardization to `ppid`.

---

## 4. Data Cleaning Steps

### Step 1 — Schema Correction
Renaming the primary identifier and disabling safe updates for bulk cleaning.
```sql
ALTER TABLE project.`messy patient data`
RENAME COLUMN pid TO ppid;

SET SQL_SAFE_UPDATES = 0;
```

### Step 2 — Standardizing Age
Converting words to numbers and stripping "years" suffixes.
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

### Step 3 — Gender & Date Cleaning
Removing corrupted "abc" text and unifying gender labels.
```sql
UPDATE project.`messy patient data`
SET Gender = CASE 
    WHEN UPPER(Gender) LIKE 'F%' THEN 'Female'
    WHEN UPPER(Gender) LIKE 'M%' THEN 'Male'
    ELSE NULL 
END;

UPDATE project.`messy patient data`
SET Check_in_Date = CASE 
    WHEN Check_in_Date LIKE '%abc' THEN REPLACE(Check_in_Date, 'abc', '')
    WHEN Check_in_Date IN ('Z04', 'Unknown', 'N/A') THEN NULL
    ELSE Check_in_Date
END;
```

### Step 4 — Clinical Data Consolidation
Mapping all Hypertension synonyms to a single category and cleaning Heart Rate entries.
```sql
UPDATE project.`messy patient data`
SET Diagnosis = CASE 
    WHEN UPPER(Diagnosis) IN ('HBP', 'H.B.P', 'ELEVATED PRESSURE', 'HIGH BLOOD PRESSURE') THEN 'Hypertension'
    WHEN Diagnosis IN ('??', '1234', 'None', 'n/a') THEN NULL
    ELSE Diagnosis 
END;

UPDATE project.`messy patient data`
SET Heart_Rate = CASE 
    WHEN Heart_Rate = 'One hundred' THEN '100'
    WHEN Heart_Rate IN ('high', 'normal', '-', 'N/A', 'NaN', '0') THEN NULL
    ELSE Heart_Rate 
END;
```

---

## 5. Summary of Changes
1.  **Identifier Updated:** `pid` successfully renamed to `ppid`.
2.  **Logic Applied:** Word-based numbers ("Forty") were successfully converted to integers.
3.  **Redundancy Removed:** Synonyms for Hypertension were merged into one group.
4.  **Junk Filtered:** All invalid strings (`abc`, `??`, `Z04`, etc.) were converted to `NULL` to ensure accurate reporting.

---

## 6. Final Verification
To ensure the data is now clean, the following audit query was executed:
```sql
SELECT ppid, Age, Gender, Check_in_Date, Diagnosis, Heart_Rate 
FROM project.`messy patient data` 
WHERE Age IS NOT NULL AND Diagnosis IS NOT NULL
LIMIT 10;
```
*Cleaned by GODWIN IBRAHIM JOHN using MySQL.*
