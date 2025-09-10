# Hospital Analytics

### 1. Introduction
**1.1 Context**  
As a data analyst at a healthcare analytics consulting firm, I have been tasked with supporting Massachusetts General Hospital in preparing their annual performance report. our analysis focuses on patient encounters, costs, insurance coverage, and behavioral trends to support planning and improve care delivery and optimizing hospital operations.

**1.2 Objective**
* Encounters Overview: Examine Trends in patient encounter volume, types and lengths
* Cost & Coverage Insights: Insurance coverage, procedures and claim costs
* Patients Behavior Analysis: visit patterns,length of stay and readmissions


### 2. About Dataset
The analysis is based on Synthetic data on ~1k patients of Massachusetts General Hospital  
Data Source: Maven Data playground  
Data Format: CSV files  
Tables: Encounters, Patients, Payers, Procedures.  
Number of records: 75592  
Number of fields: 55 unique fields across all tables


### 3. Business Questions and Solutions

**Objective 1: Encounters Overview**  
**Q1: How many total encounters occurred each year?**

<details>
<summary> Show SQL Query </summary>

```sql
SELECT * FROM payers;

SELECT 
	Year(START) AS yr,
    COUNT(Id) AS total_encounters
FROM encounters
GROUP BY yr
ORDER BY yr;
```
</details>  

**Output**

<img width="163" height="219" alt="1" src="https://github.com/user-attachments/assets/3b4f052a-bbd3-45fa-89c6-800ab73700fb" />


**Q2: For each year, what percentage of all encounters belonged to each encounter class (ambulatory, outpatient, wellness, urgent care, emergency, and inpatient)?**

<details>
<summary> Show SQL Query </summary>

```sql
SELECT 
	Year(START) AS yr,
    ROUND(SUM(CASE WHEN ENCOUNTERCLASS = "ambulatory" THEN 1 ELSE 0 END)/
    COUNT(*) * 100,1) AS ambulatory,
    ROUND(SUM(CASE WHEN ENCOUNTERCLASS = "outpatient" THEN 1 ELSE 0 END)/
    COUNT(*) * 100,1) AS outpatient,
    ROUND(SUM(CASE WHEN ENCOUNTERCLASS = "wellness" THEN 1 ELSE 0 END)/
    COUNT(*) * 100,1) AS wellness,
    ROUND(SUM(CASE WHEN ENCOUNTERCLASS = "urgentcare" THEN 1 ELSE 0 END)/
    COUNT(*) * 100,1) AS urgentcare,
    ROUND(SUM(CASE WHEN ENCOUNTERCLASS = "emergency" THEN 1 ELSE 0 END)/
    COUNT(*) * 100,1) AS emergency,
    ROUND(SUM(CASE WHEN ENCOUNTERCLASS = "inpatient" THEN 1 ELSE 0 END)/
    COUNT(*) * 100,1) AS inpatient
FROM encounters
GROUP BY yr
ORDER BY yr DESC;
```
</details>  

**Output**

<img width="415" height="215" alt="Screenshot 2025-08-27 at 8 14 40 PM" src="https://github.com/user-attachments/assets/b1cf7fdf-0654-41d3-a9c0-540207ab5316" />


**Q3: What percentage of encounters were over 24 hours versus under 24 hours?**

<details>
<summary> Show SQL Query </summary>

```sql
SELECT
	ROUND(SUM(CASE WHEN TIMESTAMPDIFF(HOUR,START,STOP) >= 24 THEN 1 ELSE 0 END)/
      COUNT(*) * 100,1) AS over_24_hours,
    ROUND(SUM(CASE WHEN TIMESTAMPDIFF(HOUR,START,STOP) < 24 THEN 1 ELSE 0 END)/
      COUNT(*) * 100,1) AS under_24_hours
FROM encounters;
```
</details>  

**Output**

<img width="215" height="55" alt="3" src="https://github.com/user-attachments/assets/b0c21579-07d7-4f4b-92e9-1aa594bc72a7" />


**Objective 2: Cost & Coverage Insights**  
**Q1:** How many encounters had zero payer coverage, and what percentage of total encounters does this represent?  

<details>
<summary> Show SQL Query </summary>

```sql
SELECT
	SUM(CASE WHEN PAYER_COVERAGE = 0 THEN 1 ELSE 0 END) AS zero_payer_coverage,
    COUNT(*) AS total_encounters,
	ROUND(SUM(CASE WHEN PAYER_COVERAGE = 0 THEN 1 ELSE 0 END)
    /COUNT(*) * 100,1) AS percent_zero_payer_coverage
FROM encounters;
```
</details>  

**Output**

<img width="405" height="54" alt="4" src="https://github.com/user-attachments/assets/8cd47331-ecf4-42e9-9060-5d85a3a2d82e" />


**Q2:** What are the top 10 most frequent procedures performed and the average base cost for each?

<details>
<summary> Show SQL Query </summary>

```sql
SELECT 
	CODE,DESCRIPTION,COUNT(*) AS num_procedures,
    ROUND(AVG(BASE_COST),1) AS avg_base_cost
FROM procedures
GROUP BY CODE,DESCRIPTION
ORDER BY num_procedures DESC
LIMIT 10;
```
</details>  

**Output**

<img width="540" height="183" alt="5" src="https://github.com/user-attachments/assets/eef9001a-1bb5-4e27-af41-c409100d472c" />

**Q3:** What are the top 10 procedures with the highest average base cost and the number of times they were performed?

<details>
<summary> Show SQL Query </summary>

```sql
SELECT 
	CODE,DESCRIPTION,COUNT(*) AS num_procedures,
    ROUND(AVG(BASE_COST),1) AS avg_base_cost
FROM procedures
GROUP BY CODE,DESCRIPTION
ORDER BY avg_base_cost DESC
LIMIT 10;
```
</details>  

**Output**

<img width="508" height="192" alt="6" src="https://github.com/user-attachments/assets/7dbe8969-4f8d-4e71-b718-921b1011ec34" />

**Q4:** What is the average total claim cost for encounters, broken down by payer?

<details>
<summary> Show SQL Query </summary>

```sql
SELECT
	p.NAME,
    ROUND(AVG(e.TOTAL_CLAIM_COST),1)AS avg_total_claim_cost
FROM payers p
LEFT JOIN encounters e
ON p.ID = e.PAYER
GROUP BY p.NAME
ORDER BY avg_total_claim_cost DESC;
```
</details>  

**Output**

<img width="265" height="186" alt="7" src="https://github.com/user-attachments/assets/70f1c051-37cf-42e9-a89e-49cea449b1e7" />


**Objective 3: Patients Behavior Analysis**  
**Q1:** How many unique patients were admitted each quarter over time?

<details>
<summary> Show SQL Query </summary>

```sql
SELECT
	CONCAT(YEAR(START), 'Q',QUARTER(START)) AS yr_qt,
    COUNT(DISTINCT PATIENT) AS num_unique_patients	 
FROM encounters
GROUP BY yr_qt;
```
</details>  

**Output**

<img width="190" height="716" alt="8" src="https://github.com/user-attachments/assets/4ba8b568-c915-4d3c-b530-8e9a2d21d660" />

**Q2:** How many patients were readmitted within 30 days of a previous encounter?

<details>
<summary> Show SQL Query </summary>

```sql
WITH cte AS (
SELECT
	PATIENT,
    START,
    STOP,
	LEAD(START)OVER(PARTITION BY PATIENT ORDER BY START) AS next_start
FROM encounters
)
SELECT COUNT(DISTINCT PATIENT) AS num_patients
FROM cte
WHERE DATEDIFF(next_start,STOP) < 30;
```
</details>  

**Output**

<img width="102" height="54" alt="9" src="https://github.com/user-attachments/assets/b0ba66eb-3999-4e82-bc4b-687a97e8be4c" />


**Q3:** Which patients had the most readmissions?

<details>
<summary> Show SQL Query </summary>

```sql
WITH cte AS (
SELECT
	PATIENT,
    START,
    STOP,
	LEAD(START)OVER(PARTITION BY PATIENT ORDER BY START) AS next_start
FROM encounters
)
SELECT 
PATIENT,
COUNT(*) AS num_readmissions
FROM cte
WHERE DATEDIFF(next_start,STOP) < 30
GROUP BY PATIENT
ORDER BY num_readmissions DESC;
```
</details>  

**Output**




<img width="340" height="241" alt="10" src="https://github.com/user-attachments/assets/4a329654-74e1-4ed1-95d7-d729c869e26c" />
