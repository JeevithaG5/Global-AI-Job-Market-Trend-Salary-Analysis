
# 🌍 Global AI Job Market Trends & Salary Insights (2025)
**SQL • Power BI • Data Engineering • Analytics Portfolio Project**  
**By: Jeevitha G**

---

## 📌 Project Overview
This project provides a full end-to-end analysis of the Global AI Job Market (2025) using:
- SQL for data cleaning, transformation & EDA
- Power BI for interactive dashboards
- DAX for KPIs and advanced measures
- Skill parsing, salary modelling, and trend analysis

The project uncovers insights about:
- Salary trends across job titles, industries, countries, experience levels
- Top-paying skills and most demanded skills
- Remote, hybrid, and onsite job adoption
- Highest-paying industries and top hiring companies
- Company-size and geography-based salary variations

This work supports:
✔ Job seekers  
✔ Hiring teams  
✔ Workforce planners  
✔ AI industry researchers

---

## 🎯 Project Objectives
1. **Data Exploration & Understanding**  
   - Analyse schema, salary fields, skill lists, and job descriptions  
   - Identify correlations between salary, experience, skills, and industry  
2. **Salary Insights Across Dimensions**  
   - Salary by job title, industry, location, experience & company size  
3. **Experience-Level & Employment Analysis**  
   - Job demand distribution by experience  
   - Hiring trends by employment type (CT / FT / PT / FL)  
4. **Work Mode Insights**  
   - Remote vs Onsite vs Hybrid  
   - Relationship between work mode and salary  
5. **Skill Demand & Revenue Insights**  
   - Most demanded skills  
   - Highest-paying skills  
   - Skill requirements by industry  
6. **Company & Industry Insights**  
   - Top hiring companies  
   - Highest-paying industries  
   - Benefits score patterns  
7. **Stakeholder Recommendations**  
   - Skills to learn  
   - Industries to target  
   - Salary benchmarking guidance

---

## 🗂️ Dataset Overview
Fields included:
```
job_id, job_title, salary_usd, salary_currency,
experience_level, employment_type, company_location,
company_size, employee_residence, remote_ratio,
required_skills, education_required, years_experience,
industry, posting_date, application_deadline,
job_description_length, benefits_score, company_name
```
Dataset was cleaned, standardized and prepared for SQL ingestion.

---

## 🧩 Data Loading & SQL Setup
**Steps**  
1. Create database: `aijobs_db`  
2. Create table: `aijobs`  
3. Import CSV using SQL Workbench Import Wizard  
4. Validate schema & data types  

**Schema Design**  
- Salary → DECIMAL(10,2)  
- Years of experience → INT  
- Dates → DATE  
- Skills → TEXT  
- Benefits Score → DECIMAL(3,1)  

---

## 🧹 Data Cleaning Pipeline
- Missing value checks  
- Standardized date formats  
- Trimmed whitespace  
- Normalized industries & country names  
- Skill-splitting using numbers table  
- Verified numeric salary_usd column  
- Removed inconsistencies  

---

## 🔄 Data Transformation Steps
- Skill extraction using `SUBSTRING_INDEX`  
- Feature engineering:
  - Skill count per job  
  - Days application is open  
  - Salary bucket classification  
- WorkMode derived from remote_ratio  
- Aggregations for:
  - Avg salary by industry  
  - Salary distribution by company size  

---

## 📐 MECE Analysis Framework
1. **Salary Analysis**: Title, Industry, Geography, Company Size, Experience  
2. **Skill Analysis**: Demand frequency, Highest-paying skills, Industry-specific top skills  
3. **Work Mode & Employment**: Remote/Hybrid/Onsite patterns, Salary differences across modes  
4. **Company & Industry Insights**: Top hiring companies, Benefit score patterns, Salary benchmarks  

---

## 🔍 Exploratory Data Analysis (SQL)
**SQL Queries Used (Core EDA & Feature Engineering)**

**1. Data Quality Checks**
```sql
SELECT COUNT(*) AS Total_Jobs FROM aijobs;

SELECT 
  SUM(CASE WHEN salary_usd IS NULL THEN 1 END) AS Missing_Salary,
  SUM(CASE WHEN required_skills IS NULL THEN 1 END) AS Missing_Skills,
  SUM(CASE WHEN posting_date IS NULL THEN 1 END) AS Missing_Posting_Date
FROM aijobs;
```
<img width="155" height="62" alt="image" src="https://github.com/user-attachments/assets/811c78eb-d910-4360-9ef7-78b9bdce0c3d" />

**2. Salary Analysis**
```sql
SELECT job_title, ROUND(AVG(salary_usd),1) AS AvgSalary
FROM aijobs
GROUP BY job_title
ORDER BY AvgSalary DESC;

SELECT industry, ROUND(AVG(salary_usd),1) AS AvgSalary
FROM aijobs
GROUP BY industry
ORDER BY AvgSalary DESC;
```
  <img width="231" height="336" alt="image" src="https://github.com/user-attachments/assets/55917d8d-2781-4e13-8146-2b2a04af064c" />

**3. Hiring Trends**
```sql
SELECT MONTH(posting_date) AS Month,
       YEAR(posting_date) AS Year,
       COUNT(*) AS TotalJobs
FROM aijobs
GROUP BY Month, Year
ORDER BY Year, Month;
```
  <img width="211" height="396" alt="image" src="https://github.com/user-attachments/assets/c5c568c7-93b6-4501-a866-0ef881d34ab4" />

**4. Experience Level %**
```sql
SELECT years_experience,
       COUNT(*) AS TotalJobs,
       CONCAT(ROUND(COUNT(*)*100/(SELECT COUNT(*) FROM aijobs),2),'%') AS PercentageJobs
FROM aijobs
GROUP BY years_experience
ORDER BY years_experience;
```

**5. Skill Parsing (Numbers Table)**
```sql
CREATE TABLE numbers (n INT PRIMARY KEY);
INSERT INTO numbers VALUES (1),(2),(3),(4),(5),(6),(7),(8),(9),(10);
```

**6. Skill Frequency**
```sql
SELECT skills, COUNT(*) AS Skill_Frequency
FROM (
  SELECT TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(required_skills, ',', n), ',', -1)) AS skills
  FROM aijobs
  JOIN numbers n
    ON CHAR_LENGTH(required_skills) - CHAR_LENGTH(REPLACE(required_skills, ',', '')) >= n.n - 1
) t
GROUP BY skills
ORDER BY Skill_Frequency DESC;
```
  <img width="264" height="378" alt="image" src="https://github.com/user-attachments/assets/dbbf2eca-d327-452c-b719-6df1c111c473" />

**7. Highest Paying Skills**
```sql
SELECT skills, ROUND(AVG(salary_usd),2) AS average_salary
FROM (
  SELECT salary_usd,
         TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(required_skills, ',', n), ',', -1)) AS skills
  FROM aijobs
  JOIN numbers n
    ON CHAR_LENGTH(required_skills) - CHAR_LENGTH(REPLACE(required_skills, ',', '')) >= n.n - 1
) t
GROUP BY skills
ORDER BY average_salary DESC;
```
  <img width="267" height="361" alt="image" src="https://github.com/user-attachments/assets/8edd115d-7d30-422b-8aa1-4fa411f7d3b0" />

**8. Top Skills by Industry**
```sql
WITH skillcount AS (
  SELECT industry,
         skills,
         COUNT(*) AS frequency
  FROM (
    SELECT industry,
           TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(required_skills, ',', n), ',', -1)) AS skills
    FROM aijobs
    JOIN numbers n
      ON CHAR_LENGTH(required_skills) - CHAR_LENGTH(REPLACE(required_skills, ',', '')) >= n.n - 1
  ) t
  GROUP BY industry, skills
),
ranked AS (
  SELECT industry, skills, frequency,
         ROW_NUMBER() OVER (PARTITION BY industry ORDER BY frequency DESC) AS rn
  FROM skillcount
)
SELECT industry, skills, frequency 
FROM ranked
WHERE rn <= 5;

```
  <img width="271" height="387" alt="image" src="https://github.com/user-attachments/assets/aa03b4c1-b525-442b-8cb2-c61d2bcb0cb9" />

---

## 📊 Power BI Dashboards
**Dashboard 1: Global AI Salary Trends**  

<img width="854" height="477" alt="image" src="https://github.com/user-attachments/assets/cbb6e6d2-d23c-4673-b54f-4b40e79483a5" />


**Dashboard 2: Skill Demand & Remote Work**  
 
<img width="866" height="479" alt="image" src="https://github.com/user-attachments/assets/4225a307-d16c-4845-8e15-0fcfb2555545" />


**Dashboard 3: Industry & Company Insights**  
  
<img width="867" height="477" alt="image" src="https://github.com/user-attachments/assets/ce129b61-e6d5-465c-ae6c-b391142ea495" />

---

## 🧮 DAX Measures Used

**Basic KPIs**  
```
Total Jobs = COUNTROWS(aijobs)
Average Salary = AVERAGE(aijobs[salary_usd])
Highest Salary = MAX(aijobs[salary_usd])
Lowest Salary = MIN(aijobs[salary_usd])
```

**Work Mode %**  
```
Hybrid% =
VAR coun = COUNTROWS(FILTER(aijob, aijob[remote_ratio] = 50))
VAR totcoun = COUNTROWS(aijob)
RETURN DIVIDE(coun, totcoun)

Onsite% =
DIVIDE(
    CALCULATE(COUNTROWS(aijob), aijob[remote_ratio] = 0),
    CALCULATE(COUNTROWS(aijob), ALL(aijob))
)

Remote% =
DIVIDE(
    CALCULATE(COUNTROWS(aijob), aijob[remote_ratio] = 100),
    CALCULATE(COUNTROWS(aijob), ALL(aijob))
)
```

**Skills KPIs**  
```
Total Unique Skills = DISTINCTCOUNT(Skills[skill])
Top Skill in Demand = 
VAR t = ADDCOLUMNS(VALUES(Skills[skill]), "SkillCount", COUNTROWS(Skills))
VAR top1 = TOPN(1, t, [SkillCount], DESC)
RETURN CONCATENATEX(top1, Skills[skill], ", ")

Top Paying Skill = 
VAR t = ADDCOLUMNS(SUMMARIZE(Skills, Skills[skill]), "AvgSalary", CALCULATE(AVERAGE(aijobs[salary_usd])))
VAR top1 = TOPN(1, t, [AvgSalary], DESC)
RETURN CONCATENATEX(top1, Skills[skill], ", ")
```

**Company & Industry KPIs**: Top Paying Company, Top Hiring Industry, Top Hiring Country, Top Company Size, Top Education Required  

---

## 🧠 Key Insights
- **Top Paying Industries**: Consulting, Manufacturing, Tech Services  
- **Most Demanded Skills**: Python, SQL, Kubernetes, Machine Learning, Deep Learning  
- **Highest-Paying Skills**: Deep Learning, NLP, AWS, Java, Hadoop  
- **Remote Work**: ~33% Remote, Hybrid dominant  
- **Salary Distribution**: Highest ~$131K, Median ~$60K, Lowest ~$33K  
- **Skill Intensity**: Senior roles require 7–12 skills on average  

---

## ✅ Conclusion
This project demonstrates **end-to-end analytics** using SQL, Power BI, and DAX. Insights cover: salary predictors, skill demand, industry trends, remote work impact, and geographic patterns — making it a strong portfolio showcase.

---

## 📘 Significance of the Project
Helps job seekers, companies, policymakers, and analysts understand skill requirements, salary benchmarking, and global AI workforce trends.

---

