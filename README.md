# Healthcare-No-Show-Project


# 📊 Healthcare No-Show Analysis – SQL Cleaning Script

This project focuses on preparing and transforming raw patient appointment data for effective analysis. The dataset contains patient demographics, appointment dates, and whether the patient showed up. Cleaning was done using **SQL**, and the result is a new table: `noshow_cleaned`.

---

##  Cleaning Objectives Using SQL

| Step | Description |
|------|-------------|
|  Rename Columns | Standardized column names for clarity |
|  Format Dates | Converted date columns to `YYYY-MM-DD` using `strftime()` |
|  Age Banding | Created age groups: `0–18`, `19–30`, `31–45`, `46–60`, `61+` |
|  Boolean Conversion | Changed `TRUE`/`FALSE` into human-readable `Yes`/`No` |
|  SMS Status | Renamed and recoded `SMS_received` to `Got SMS` / `No SMS` |
|  Show Status | Inverted `No-show` to a clearer `Showed Up` / `No-Show` label |
| ⏱ Wait Time | Added `Wait_Days` column by subtracting `ScheduledDay` from `AppointmentDay` |

---

## 🧠 SQL Script Used

```sql
CREATE TABLE noshow_cleaned AS
SELECT
  PatientId,
  AppointmentID,
  Gender,
  strftime('%Y-%m-%d', ScheduledDay) AS ScheduledDay,
  strftime('%Y-%m-%d', AppointmentDay) AS AppointmentDay,
  Age,

  CASE
    WHEN Age BETWEEN 0 AND 18 THEN '0–18'
    WHEN Age BETWEEN 19 AND 30 THEN '19–30'
    WHEN Age BETWEEN 31 AND 45 THEN '31–45'
    WHEN Age BETWEEN 46 AND 60 THEN '46–60'
    ELSE '61+'
  END AS Age_Band,

  Neighbourhood,

  CASE WHEN Scholarship = 'TRUE' THEN 'Yes' ELSE 'No' END AS Scholarship,
  CASE WHEN Hipertension = 'TRUE' THEN 'Yes' ELSE 'No' END AS Hypertension,
  CASE WHEN Diabetes = 'TRUE' THEN 'Yes' ELSE 'No' END AS Diabetes,
  CASE WHEN Alcoholism = 'TRUE' THEN 'Yes' ELSE 'No' END AS Alcoholism,
  CASE WHEN Handcap = 'TRUE' THEN 'Yes' ELSE 'No' END AS Handicap,

  CASE WHEN SMS_received = 'TRUE' THEN 'Got SMS' ELSE 'No SMS' END AS SMS_Status,
  CASE WHEN "No-show" = 'No' THEN 'Showed Up' ELSE 'No-Show' END AS Show_Status,

  julianday(AppointmentDay) - julianday(ScheduledDay) AS Wait_Days

FROM noshow_raw;
