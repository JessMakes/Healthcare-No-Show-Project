## Healthcare No-Show Insights

### Project Overview

This project analyzes a real-world healthcare dataset of over 110,000 patient appointments from Brazil, sourced from Kaggle. The goal was to uncover patterns behind missed appointments (no-shows) 
and generate actionable strategies to improve patient attendance.

Key variables explored include:
- Patient age and gender  
- Chronic medical conditions  
- Whether an SMS reminder was received  

---

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

In a separate transformation exercise, I used  **Excel** and **Power Query** to reshape the data for condition-level analysis and visualization in Power BI.

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


I further explored the dataset using the SQL. 


```sql
1.Total Appointments Scheduled
SELECT COUNT(*) AS total_appointments
FROM noshow_cleaned;

This returns the total number of appointment records in the dataset.
106,987 appointments were scheduled in total.


2.No-Show by Age Band
SELECT
  CASE
    WHEN Age BETWEEN 0 AND 18 THEN '0–18'
    WHEN Age BETWEEN 19 AND 30 THEN '19–30'
    WHEN Age BETWEEN 31 AND 45 THEN '31–45'
    WHEN Age BETWEEN 46 AND 60 THEN '46–60'
    ELSE '61+'
  END AS Age_Band,
  COUNT(*) AS total_appointments,
  SUM(CASE WHEN Show_Status = 'No-Show' THEN 1 ELSE 0 END) AS no_shows
FROM noshow_cleaned
GROUP BY Age_Band
ORDER BY no_shows DESC;

### Insight: Highest No-Shows in the 0–18 Age Band

The 0–18 age group recorded the highest number of no-shows (5,708 out of 25,327 appointments). This is likely due to
parental or guardian-related factors, such as:

- Forgetfulness or scheduling conflicts
- Lower perceived urgency if the child shows no severe symptoms
- School timing clashes
- Transportation or financial challenges

**Implication:** Healthcare providers may consider targeted SMS reminders for pediatric appointments or follow-up calls to guardians.


3.Attendance by Chronic Condition
This query calculates the total number of patients with each chronic condition (Hypertension, Diabetes, Alcoholism, Handicap) using
separate `SELECT` statements combined with `UNION ALL`.

Purpose of Using `UNION ALL`:
The dataset stores each chronic condition as a separate column. To count how many patients have each condition, we must query each column independently. 
Rather than running four separate queries, I used `UNION ALL` to stack the result vertically to get the final result.



SELECT 'Hypertension' AS Condition, COUNT(*) AS count
FROM noshow_cleaned WHERE Hypertension = 'Yes'

UNION ALL

SELECT 'Diabetes', COUNT(*) 
FROM noshow_cleaned WHERE Diabetes = 'Yes'

UNION ALL

SELECT 'Alcoholism', COUNT(*) 
FROM noshow_cleaned WHERE Alcoholism = 'Yes'

UNION ALL

SELECT 'Handicap', COUNT(*) 
FROM noshow_cleaned WHERE Handicap = 'Yes';

Finding:
Hypertension had the highest appointment count among chronic conditions, followed by diabetes.
| Hypertension patients showed the most no-shows | Patients managing chronic illnesses like hypertension are at higher risk of missing appointments. |


4.Total Appointments by SMS Status
"How many patients actually received a reminder versus those who didn’t?"

SELECT SMS_STATUS,
       COUNT(*) AS total_appointments
FROM noshow_cleaned
GROUP BY SMS_Status;

Finding:
Only 32% of the 106,987 scheduled appointments received an SMS reminder, totaling 34,585 patients.

5.No-Show Count by SMS Status

SELECT SMS_Status,
       SUM(CASE WHEN Show_Status = 'No-Show' THEN 1 ELSE 0 END) AS no_shows
FROM noshow_cleaned
GROUP BY SMS_Status;

Finding:
This compares no-shows between patients who did and didn’t receive SMS.
Patients who received SMS reminders had significantly fewer no-shows, with an estimated 20% improvement in attendance.

6.Do attendance patterns differ between genders?

SELECT Gender,
       COUNT(*) AS total_appointments,
       SUM(CASE WHEN Show_Status = 'Showed Up' THEN 1 ELSE 0 END) AS showed_up
FROM noshow_cleaned
GROUP BY Gender;

Finding:
Women attended appointments more consistently than men, representing roughly 66% of total appointments.


Key Insights:

| Insight | Summary |
|--------|---------|


| Only about 40% of patients received an SMS reminder | SMS reminders were underused but had a clear positive effect. |
| SMS reminders reduced no-shows by approximately 20% | Patients who received SMS reminders were significantly more likely to attend. |
| Women were more likely to attend than men | Approximately 66% of attendees were female, suggesting potential for gender-specific outreach. |

---

Recommendations

- Expand SMS coverage to all patients, as reminders are linked to improved attendance.
- Implement receptionist follow-ups or calls for patients with chronic conditions, especially hypertension.
- Explore gender-based engagement strategies to improve male attendance rates.

---

This project demonstrates how data-driven strategies can support healthcare providers in reducing no-shows, optimizing resources, and improving overall patient care.


Tools Used

- **Excel**: Initial data cleaning and column derivation (e.g., labeling SMS status, creating age bands)
- **Power Query**: Data reshaping from wide to long format for condition-level analysis
- **SQL**: Calculated aggregated metrics, conditional counts, and implemented grouping logic
- **Power BI**: Built interactive dashboards and developed insights based on the transformed dataset

---

Dataset Source

**Dataset**: Brazil No-Show Appointments  
**Source**: [Kaggle – Joni Arroba](https://www.kaggle.com/datasets/joniarroba/noshowappointments)  
**Size**: 110,000+ patient appointment records  
**Usage**: For educational and non-commercial analysis purposes only

---

Financial Impact of Missed Appointments

- **United States**:  
  - Estimated **$150 billion** in annual losses  
  - Roughly **$200 per missed visit**

- **Australia**:  
  - **$500,000/year** in lost revenue at St Vincent’s Hospital  
  - Up to **$3.8 million/month** in losses reported in QLD outpatient clinics

**Sources**: NSW Government, The Guardian, TransLoc








---



