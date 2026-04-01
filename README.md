MESSY PATIENT DATA CLEANING

Identified Data Errors before data cleaning
. Inconsistent Age Formats: Mixed numeric values (41), words (twenty-two, forty), units (85 years), and invalid entries (0, -5, unknown).
. Inconsistent Gender: Mixed casing (Female, FEMALE, male) and missing values.
. Messy Dates: Dates are in multiple formats: Excel serial numbers (44819), long strings (Saturday, Oct 21...), ISO format (2024-01-24), and garbage text (Z04, abc).
. Non-Standardized Diagnosis: Multiple names for the same condition (HBP, H.B.P, High blood pressure, Elevated Pressure).
. Invalid Heart Rate: Mixed numeric values with text (One hundred, high, normal) and symbols (-).

1. TO VIEW THE TABLE BEFORE CLEANING
SELECT * FROM project.`messy patient data`;

2. RENAMING THE COLUNM ID
ALTER TABLE project.`messy patient data`
Rename column pid TO ppid;

3. THE UNLOCK STEP; turns off a "safety lock."
SET SQL_SAFE_UPDATES = 0;

4. CLEANING THE AGE; Converts words like "Forty" to "40," removes "years" from numbers, and deletes impossible ages like "0" or "-5."
UPDATE project.`messy patient data`
SET Age = CASE 
    WHEN Age LIKE '%years%' THEN TRIM(REPLACE(Age, 'years', ''))
    WHEN UPPER(Age) IN ('TWENTY-TWO', 'TWENTY TWO') THEN '22'
    WHEN UPPER(Age) = 'TWENTY FIVE' THEN '25'
    WHEN UPPER(Age) = 'FORTY' THEN '40'
    WHEN Age IN ('0', '-5', 'unknown', 'NaN', 'N/A') THEN NULL
    ELSE Age 
END;

6. CLEANING GENDER: If gender starts with an "F" (like female, FEMALE, or f), it changes it to a standard "Female". If it starts with an "M", it becomes "Male". Everything else is cleared out.
UPDATE project.`messy patient data`
SET Gender = CASE 
    WHEN UPPER(Gender) LIKE 'F%' THEN 'Female'
    WHEN UPPER(Gender) LIKE 'M%' THEN 'Male'
    ELSE NULL 
END;

8. CLEANING DATE: Looks for "abc" accidentally typed at the end of dates (like 2021-03-14abc) and deletes the "abc" and removes "Z04" or "Unknown," which are codes that aren't actual dates.   
UPDATE project.`messy patient data`
SET Check_in_Date = CASE 
    WHEN Check_in_Date LIKE '%abc' THEN REPLACE(Check_in_Date, 'abc', '')
    WHEN Check_in_Date IN ('Z04', 'Unknown', 'N/A') THEN NULL
    ELSE Check_in_Date
END;

9. Standardizing the Diagnosis;renaming "HBP," "H.B.P," and "High Blood Pressure" and renames them all to one official term: "Hypertension." and also deletes junk like "??" or "1234."
UPDATE project.`messy patient data`
SET Diagnosis = CASE 
    WHEN UPPER(Diagnosis) IN ('HBP', 'H.B.P', 'ELEVATED PRESSURE', 'HIGH BLOOD PRESSURE') THEN 'Hypertension'
    WHEN Diagnosis IN ('??', '1234', 'None', 'n/a') THEN NULL
    ELSE Diagnosis 
END;

10. Cleaning Heart Rate; changing the words "One hundred" to the number "100." and deletes non-numeric descriptions like "high," "normal," or dashes ("-") so the column stays strictly numeric.
UPDATE project.`messy patient data`
SET Heart_Rate = CASE 
    WHEN Heart_Rate = 'One hundred' THEN '100'
    WHEN Heart_Rate IN ('high', 'normal', '-', 'N/A', 'NaN', '0') THEN NULL
    ELSE Heart_Rate 
END;



Data cleaning 
