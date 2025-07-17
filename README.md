# Healthcare-No-Show-Project

###  Data Cleaning and Transformation (SQL + Power Query)

The no-show patient dataset was cleaned and transformed using **SQL**, **Excel**, and **Power Query**. These tools were applied on separate occasions to demonstrate flexibility 
in using both code-first and visual transformation methods.

---

####  SQL-Based Cleaning

The following transformations were implemented using **SQLite** to create the cleaned table: `noshow_cleaned`.

| Task                   | Columns Affected                        | Description                                                                  |
|------------------------|-----------------------------------------|------------------------------------------------------------------------------|
| Format Dates           | ScheduledDay, AppointmentDay            | Converted to `YYYY-MM-DD` using `strftime()`                                |
| Create Age Groups      | Age → Age_Band                          | Grouped into: `0–18`, `19–30`, `31–45`, `46–60`, `61+`                       |
| Convert Booleans       | Scholarship, Hypertension, etc.         | Converted `TRUE` → `"Yes"`, `FALSE` → `"No"`                                 |
| Relabel SMS Received   | SMS_received → SMS_Status               | `"TRUE"` → `"Got SMS"`, `"FALSE"` → `"No SMS"`                              |
| Relabel Showed_Up      | No-show → Show_Status                   | `"No"` → `"Showed Up"`, `"Yes"` → `"No-Show"`                               |
| Add Wait Time          | ScheduledDay, AppointmentDay → Wait_Days| Calculated wait time using `julianday()` difference                         |

---

####  Power Query Cleaning (Power BI)

In a separate transformation exercise, **Excel** and **Power Query** were used to reshape the data for condition-level analysis and visualization.

| Task                      | Tool Used   | Columns Affected                    | Description                                                                 |
|---------------------------|-------------|-------------------------------------|-----------------------------------------------------------------------------|
| Label SMS Status          | Excel       | SMS_received → SMS_Status           | Converted binary: TRUE → `"Got SMS"`, FALSE → `"No SMS"`                   |
| Simplify No-show Logic    | Excel       | No-show → Showed_Up                 | `"No"` → `"Showed Up"`, `"Yes"` → `"No-Show"`                              |
| Create Age Groups         | Excel       | Age → Age Band                      | Banded into: `0–18`, `19–30`, `31–45`, `46–60`, `61+`                       |
| Unpivot Conditions        | Power Query | Diabetes, Hypertension, Alcoholism  | Unpivoted into a single `Condition` column                                 |
| Add Presence Indicator    | Power Query | Condition → Present                 | Boolean column to indicate presence of condition                           |
| Filter Relevant Records   | Power Query | Present                             | Filtered for `Present = TRUE` only                                         |
| Relabel Attendance Status | Power Query | Showed_Up                           | Converted TRUE → `"Showed Up"`, FALSE → `"No-Show"`                        |

---

###  Why This Matters

Using **SQL** enabled backend data preparation with precision and performance — ideal for scalable analysis and BI reporting. Meanwhile,

**Power Query** and **Excel** allowed for rapid reshaping and intuitive, no-code data exploration.

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
