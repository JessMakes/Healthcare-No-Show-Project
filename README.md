## Healthcare No-Show Insights

### Project Overview

This project analyzes a real-world healthcare dataset of over 110,000 patient appointments from Brazil, sourced from Kaggle. The goal was to uncover patterns behind missed appointments (no-shows) 
and generate actionable strategies to improve patient attendance.

Key variables explored include:
- Patient age and gender  
- Chronic medical conditions  
- Whether an SMS reminder was received  

The analysis was conducted using Excel, Power Query, Power BI, and SQL for data transformation, visualization, and insight development.

The findings are supported by external research showing the high cost of no-shows:
No-shows cost the U.S. healthcare system approximately $150 billion annually, with millions in lost revenue for Australian hospitals.

---


```sql
1.Total Appointments Scheduled
SELECT COUNT(*) AS total_appointments
FROM noshow_raw;

Answer:
This returns the total number of appointment records in the dataset.
110,527 appointments were scheduled in total.


2.No-Show by Age Band
SELECT
  CASE
    WHEN Age BETWEEN 0 AND 17 THEN 'Under 18'
    WHEN Age BETWEEN 18 AND 40 THEN '18-40'
    WHEN Age BETWEEN 41 AND 65 THEN '41-65'
    ELSE '66+'
  END AS Age_Band,
  COUNT(*) AS total_appointments,
  SUM(CASE WHEN Showed_up = 'FALSE' THEN 1 ELSE 0 END) AS no_shows
FROM noshow_raw
GROUP BY Age_Band
ORDER BY no_shows DESC;

Answer:
This returns the total number of appointment records in the dataset.
110,527 appointments were scheduled in total.

3.Attendance by Chronic Condition
SELECT 'Hypertension' AS Condition, COUNT(*) AS count
FROM noshow_raw WHERE Hipertension = 'TRUE'
UNION ALL
SELECT 'Diabetes', COUNT(*) FROM noshow_raw WHERE Diabetes = 'TRUE'
UNION ALL
SELECT 'Alcoholism', COUNT(*) FROM noshow_raw WHERE Alcoholism = 'TRUE'
UNION ALL
SELECT 'Handcap', COUNT(*) FROM noshow_raw WHERE Handcap = 'TRUE';

Answer:
This query combines results from different chronic condition flags.
 Hypertension had the highest appointment count among chronic conditions, followed by diabetes.


4.Total Appointments by SMS Status
SELECT SMS_received,
       COUNT(*) AS total_appointments
FROM noshow_raw
GROUP BY SMS_received;


5.No-Show Count by SMS Status
SELECT SMS_received,
       SUM(CASE WHEN Showed_up = 'FALSE' THEN 1 ELSE 0 END) AS no_shows
FROM noshow_raw
GROUP BY SMS_received;

Answer:
This compares no-shows between patients who did and didn’t receive SMS.
Patients who received SMS reminders had significantly fewer no-shows, with an estimated 20% improvement in attendance.

6.Do attendance patterns differ between genders?
Showed Up Status by Gender
SELECT Gender,
       COUNT(*) AS total_appointments,
       SUM(CASE WHEN Showed_up = 'TRUE' THEN 1 ELSE 0 END) AS showed_up
FROM noshow_raw
GROUP BY Gender;

Answer:
This breaks down attendance behavior by gender.
Women attended appointments more consistently than men, representing roughly 66% of total

### Key Insights from the Dashboard

| Insight | Summary |
|--------|---------|
| Ages 41–65 had the highest no-show volume | This middle-aged group accounted for the greatest number of missed appointments, making them a high-priority segment for intervention. |
| Hypertension patients showed the most no-shows | Patients managing chronic illnesses like hypertension are at higher risk of missing appointments. |
| Only about 40% of patients received an SMS reminder | SMS reminders were underused but had a clear positive effect. |
| SMS reminders reduced no-shows by approximately 20% | Patients who received SMS reminders were significantly more likely to attend. |
| Women were more likely to attend than men | Approximately 66% of attendees were female, suggesting potential for gender-specific outreach. |

---

### Recommendations

- Expand SMS coverage to all patients, as reminders are linked to improved attendance.
- Target patients aged 41–65 with additional or repeated reminders.
- Implement receptionist follow-ups or calls for patients with chronic conditions, especially hypertension.
- Explore gender-based engagement strategies to improve male attendance rates.

---

This project demonstrates how data-driven strategies can support healthcare providers in reducing no-shows, optimizing resources, and improving overall patient care.


## Tools Used

- **Excel**: Initial data cleaning and column derivation (e.g., labeling SMS status, creating age bands)
- **Power Query**: Data reshaping from wide to long format for condition-level analysis
- **SQL**: Calculated aggregated metrics, conditional counts, and implemented grouping logic
- **Power BI**: Built interactive dashboards and developed insights based on the transformed dataset

---

## Dataset Source

**Dataset**: Brazil No-Show Appointments  
**Source**: [Kaggle – Joni Arroba](https://www.kaggle.com/datasets/joniarroba/noshowappointments)  
**Size**: 110,000+ patient appointment records  
**Usage**: For educational and non-commercial analysis purposes only

---

## Financial Impact of Missed Appointments

- **United States**:  
  - Estimated **$150 billion** in annual losses  
  - Roughly **$200 per missed visit**

- **Australia**:  
  - **$500,000/year** in lost revenue at St Vincent’s Hospital  
  - Up to **$3.8 million/month** in losses reported in QLD outpatient clinics

**Sources**: NSW Government, The Guardian, TransLoc














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
