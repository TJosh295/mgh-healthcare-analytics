# Clinical Operations & Financial Performance Dashboard
**Project 2: Healthcare Analytics Case Study**

## 📌 Project Overview
This project provides an end-to-end data analysis of historical patient records from Massachusetts General Hospital (2011–2022). Operating as a consultant analyst, the goal was to investigate chronological patient admission trends, calculate operational efficiency (Length of Stay), and evaluate the financial relationship between hospital encounter costs and insurance provider coverage.

## 🛠️ Tech Stack & Methodology
* **Database Management:** MySQL Workbench (Data structure creation, aggregation, and precise metric validation).
* **Business Intelligence:** Power BI Desktop (Star-schema relational modeling, DAX measure engineering, and interactive dashboard design).
* **Data Architecture:** Designed a **Star Schema** relational model connecting multiple dimension tables (`patients`, `payers`, `procedures`) to a central fact table (`encounters`).

---

## 📐 Phase 1: Analytical SQL Queries
Before building visuals, the database logic and key metrics were validated inside MySQL Workbench using precise time-interval logic.
![SQL Workbench Queries](Project%202%20Query.png)

### 1. Patient Admissions & Readmissions Over Time
*Identified historical patterns and sequential visits utilizing SQL window functions.*
```sql
WITH RankedVisits AS (
    SELECT 
        Id AS encounter_id,
        PATIENT AS patient_id,
        START AS admission_date,
        YEAR(START) AS admission_year,
        ROW_NUMBER() OVER (PARTITION BY PATIENT ORDER BY START) AS visit_number
    FROM encounters
)
SELECT 
    admission_year,
    COUNT(encounter_id) AS total_admissions,
    SUM(CASE WHEN visit_number > 1 THEN 1 ELSE 0 END) AS total_readmissions,
    ROUND((SUM(CASE WHEN visit_number > 1 THEN 1 ELSE 0 END) / COUNT(encounter_id)) * 100, 2) AS readmission_rate_pct
FROM RankedVisits
GROUP BY admission_year
ORDER BY admission_year;
```

### 2. Clinical Operational Efficiency (Precise Length of Stay)
*Calculated stay durations by transforming minute intervals into accurate hour and day metrics to avoid fractional truncation issues.*
```sql
SELECT 
    e.ENCOUNTERCLASS AS visit_type,
    COUNT(e.Id) AS total_visits,
    ROUND(AVG(TIMESTAMPDIFF(MINUTE, e.START, e.STOP) / 60.0), 2) AS avg_stay_hours,
    ROUND(AVG(TIMESTAMPDIFF(MINUTE, e.START, e.STOP) / 1440.0), 2) AS avg_stay_days
FROM encounters e
GROUP BY e.ENCOUNTERCLASS
ORDER BY avg_stay_hours DESC;
```

### 3. Financial Costs and Subsidized Care Analysis
*Evaluated base charges against claims coverage to assess financial exposure.*
```sql
SELECT 
    ROUND(AVG(e.TOTAL_CLAIM_COST), 2) AS global_avg_cost_per_visit,
    ROUND(AVG(e.PAYER_COVERAGE), 2) AS global_avg_insurance_covered,
    ROUND(AVG(e.TOTAL_CLAIM_COST - e.PAYER_COVERAGE), 2) AS global_avg_patient_out_of_pocket,
    (SELECT COUNT(p.CODE) FROM procedures p JOIN encounters enc ON p.ENCOUNTER = enc.Id WHERE enc.PAYER_COVERAGE > 0) AS total_procedures_covered
FROM encounters e;
```

---

## 📊 Phase 2: Power BI Implementation Process
After standardizing the relational backend, the live local MySQL server instance was securely connected directly to Power BI Desktop.

1. **Relational Modeling:** Established active, manual `One-to-Many (1:*)` relationships between dimensions and facts using shared identity strings (`Id -> PATIENT`, `Id -> PAYER`, `ENCOUNTER -> Id`).
2. **Data Type Standardization:** Converted raw textual timestamps into structured `Date/Time` elements to support chronological filters.
3. **DAX Measure Engineering:** Formulated dedicated metrics to populate high-level dashboard elements dynamically:
   * **Average Stay Duration:** 
     `Avg Stay Hours = AVERAGEX('hospital_db encounters', DATEDIFF('hospital_db encounters'[START], 'hospital_db encounters'[STOP], MINUTE) / 60.0)`
![Power BI Star Schema Model](Project%202%20Data%20Model.png)
   * **Subsidized Performance:** 
     `Procedures Covered = CALCULATE(COUNT('hospital_db procedures'[CODE]), 'hospital_db encounters'[PAYER_COVERAGE] > 0)`

---

## 💡 Executive Insights & Strategic Outcomes
* **Operational Volumes:** The hospital successfully managed **27.891K total admissions** over the tracked timeline. Patient volumes peaked heavily during the 2019–2020 period before stabilizing, highlighting critical historical surge periods.
* **Bed & Care Efficiency:** Across all clinical departments, the global average length of a hospital stay stands tightly at **7.27 hours**, heavily influenced by rapid-turnaround wellness checkups and urgent care operations.
* **Financial Footprint:** Inpatient treatments present a vastly disproportionate cost footprint relative to alternative clinical classes. However, out-of-pocket patient risk remains low for everyday procedures, as the overarching majority of medical practices  are absorbed seamlessly by insurance infrastructurabsorbed seamlessly by insurance infrastructure.
*## 🖥️ Dashboard Visualization
## 🖥️ Dashboard Visualization
![Dashboard Workspace](Project%202%20Dashboard.png)
