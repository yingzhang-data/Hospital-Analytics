# Hospital-Analytics

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
	COUNT(TIMESTAMPDIFF(HOUR,START,STOP))
FROM encounters
WHERE TIMESTAMPDIFF(HOUR,START,STOP) >= 24;

SELECT
	COUNT(TIMESTAMPDIFF(HOUR,START,STOP))
FROM encounters
WHERE TIMESTAMPDIFF(HOUR,START,STOP) < 24;

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


Objective 2: Cost & Coverage Insights  
* How many encounters had zero payer coverage, and what percentage of total encounters does this represent?
* What are the top 10 most frequent procedures performed and the average base cost for each?
* What are the top 10 procedures with the highest average base cost and the number of times they were performed?
* What is the average total claim cost for encounters, broken down by payer?

Objective 3: Patients Behavior Analysis  
* How many unique patients were admitted each quarter over time?
* How many patients were readmitted within 30 days of a previous encounter?
* Which patients had the most readmissions?
