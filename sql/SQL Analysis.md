# HR Analytics --- SQL Analysis

## Project Overview

This SQL analysis uses the HR employee dataset to answer common Human
Resources questions related to:

-   Employee count
-   Attrition
-   Attrition rate
-   Active employees
-   Average age
-   Gender-wise attrition
-   Department-wise attrition
-   Age-group analysis
-   Education-field-wise attrition
-   Gender and age-group attrition
-   Job satisfaction by job role

**Database:** PostgreSQL\
**Table:** `hrdata`

------------------------------------------------------------------------

## 1. Create the HR Data Table

``` sql
CREATE TABLE hrdata (
    emp_no INT PRIMARY KEY,
    gender VARCHAR(50) NOT NULL,
    marital_status VARCHAR(50),
    age_band VARCHAR(50),
    age INT,
    department VARCHAR(50),
    education VARCHAR(100),
    education_field VARCHAR(100),
    job_role VARCHAR(100),
    business_travel VARCHAR(50),
    employee_count INT,
    attrition VARCHAR(10),
    attrition_label VARCHAR(50),
    job_satisfaction INT,
    active_employee INT
);
```

### Column Purpose

  Column               Description
  -------------------- ------------------------------------------------
  `emp_no`             Unique employee ID
  `gender`             Employee gender
  `marital_status`     Marital status
  `age_band`           Employee age group
  `age`                Employee age
  `department`         Department
  `education`          Education level
  `education_field`    Education field
  `job_role`           Job role
  `business_travel`    Business travel frequency
  `employee_count`     Employee count value
  `attrition`          Whether the employee left
  `attrition_label`    Current or former employee
  `job_satisfaction`   Job satisfaction rating from 1--4
  `active_employee`    1 for active employees, 0 for former employees

------------------------------------------------------------------------

## 2. Import the CSV Data

The original project used a local Windows path. For a GitHub project,
avoid hard-coded personal paths.

Example PostgreSQL command:

``` sql
COPY hrdata
FROM 'path/to/hrdata.csv'
DELIMITER ','
CSV HEADER;
```

> Replace the path with the actual location of `hrdata.csv` on your
> computer.

------------------------------------------------------------------------

# HR Analysis Queries

## 3. Employee Count

**Question:** How many employees are in the dataset?

``` sql
SELECT SUM(employee_count) AS employee_count
FROM hrdata;
```

------------------------------------------------------------------------

## 4. Attrition Count

**Question:** How many employees have left the company?

``` sql
SELECT COUNT(*) AS attrition_count
FROM hrdata
WHERE attrition = 'Yes';
```

------------------------------------------------------------------------

## 5. Attrition Rate

**Question:** What percentage of employees have left?

``` sql
SELECT
    ROUND(
        COUNT(*) FILTER (WHERE attrition = 'Yes')::NUMERIC
        / SUM(employee_count) * 100,
        2
    ) AS attrition_rate
FROM hrdata;
```

------------------------------------------------------------------------

## 6. Active Employees

**Question:** How many employees are currently active?

``` sql
SELECT SUM(employee_count) AS active_employees
FROM hrdata
WHERE attrition = 'No';
```

------------------------------------------------------------------------

## 7. Average Age

**Question:** What is the average age of employees?

``` sql
SELECT ROUND(AVG(age), 0) AS average_age
FROM hrdata;
```

------------------------------------------------------------------------

## 8. Attrition by Gender

**Question:** How many employees left from each gender?

``` sql
SELECT
    gender,
    COUNT(*) AS attrition_count
FROM hrdata
WHERE attrition = 'Yes'
GROUP BY gender
ORDER BY attrition_count DESC;
```

------------------------------------------------------------------------

## 9. Department-wise Attrition

**Question:** Which departments have the highest number of employees
leaving?

``` sql
SELECT
    department,
    COUNT(*) AS attrition_count,
    ROUND(
        COUNT(*)::NUMERIC
        / (SELECT COUNT(*) FROM hrdata WHERE attrition = 'Yes') * 100,
        2
    ) AS attrition_percentage
FROM hrdata
WHERE attrition = 'Yes'
GROUP BY department
ORDER BY attrition_count DESC;
```

------------------------------------------------------------------------

## 10. Employees by Age

**Question:** How many employees belong to each age?

``` sql
SELECT
    age,
    SUM(employee_count) AS employee_count
FROM hrdata
GROUP BY age
ORDER BY age;
```

------------------------------------------------------------------------

## 11. Attrition by Education Field

**Question:** Which education fields have the highest attrition?

``` sql
SELECT
    education_field,
    COUNT(*) AS attrition_count
FROM hrdata
WHERE attrition = 'Yes'
GROUP BY education_field
ORDER BY attrition_count DESC;
```

------------------------------------------------------------------------

## 12. Attrition by Age Group and Gender

**Question:** How is attrition distributed across age groups and gender?

``` sql
SELECT
    age_band,
    gender,
    COUNT(*) AS attrition_count,
    ROUND(
        COUNT(*)::NUMERIC
        / (SELECT COUNT(*) FROM hrdata WHERE attrition = 'Yes') * 100,
        2
    ) AS attrition_percentage
FROM hrdata
WHERE attrition = 'Yes'
GROUP BY age_band, gender
ORDER BY age_band, gender;
```

------------------------------------------------------------------------

## 13. Job Satisfaction by Job Role

**Question:** How are job satisfaction ratings distributed across
different job roles?

PostgreSQL's `tablefunc` extension can be used to create a
cross-tabulation.

### Enable the extension

``` sql
CREATE EXTENSION IF NOT EXISTS tablefunc;
```

### Create the cross-tabulation

``` sql
SELECT *
FROM crosstab(
    $$
    SELECT
        job_role,
        job_satisfaction,
        SUM(employee_count)
    FROM hrdata
    GROUP BY job_role, job_satisfaction
    ORDER BY job_role, job_satisfaction
    $$
) AS ct (
    job_role VARCHAR(100),
    rating_1 NUMERIC,
    rating_2 NUMERIC,
    rating_3 NUMERIC,
    rating_4 NUMERIC
)
ORDER BY job_role;
```

### Rating Meaning

    Rating Meaning
  -------- --------------------------
         1 Low satisfaction
         2 Medium-low satisfaction
         3 Medium-high satisfaction
         4 High satisfaction

------------------------------------------------------------------------

# Additional Useful Queries

## 14. Employees by Department

``` sql
SELECT
    department,
    COUNT(*) AS employee_count
FROM hrdata
GROUP BY department
ORDER BY employee_count DESC;
```

## 15. Employees by Job Role

``` sql
SELECT
    job_role,
    COUNT(*) AS employee_count
FROM hrdata
GROUP BY job_role
ORDER BY employee_count DESC;
```

## 16. Attrition by Job Role

``` sql
SELECT
    job_role,
    COUNT(*) AS attrition_count
FROM hrdata
WHERE attrition = 'Yes'
GROUP BY job_role
ORDER BY attrition_count DESC;
```

## 17. Attrition by Age Band

``` sql
SELECT
    age_band,
    COUNT(*) AS attrition_count
FROM hrdata
WHERE attrition = 'Yes'
GROUP BY age_band
ORDER BY attrition_count DESC;
```

## 18. Attrition by Business Travel

``` sql
SELECT
    business_travel,
    COUNT(*) AS attrition_count
FROM hrdata
WHERE attrition = 'Yes'
GROUP BY business_travel
ORDER BY attrition_count DESC;
```

## 19. Attrition by Marital Status

``` sql
SELECT
    marital_status,
    COUNT(*) AS attrition_count
FROM hrdata
WHERE attrition = 'Yes'
GROUP BY marital_status
ORDER BY attrition_count DESC;
```

## 20. Attrition by Education Level

``` sql
SELECT
    education,
    COUNT(*) AS attrition_count
FROM hrdata
WHERE attrition = 'Yes'
GROUP BY education
ORDER BY attrition_count DESC;
```

------------------------------------------------------------------------

## SQL Skills Demonstrated

-   `CREATE TABLE`
-   `COPY`
-   `SELECT`
-   `WHERE`
-   `GROUP BY`
-   `ORDER BY`
-   `COUNT()`
-   `SUM()`
-   `AVG()`
-   `ROUND()`
-   `FILTER`
-   Subqueries
-   Type casting
-   Conditional analysis
-   Cross-tabulation with PostgreSQL `tablefunc`
