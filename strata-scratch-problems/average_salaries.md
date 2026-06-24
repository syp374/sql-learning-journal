# Average Salaries

**Source:** StrataScratch

**Link:** https://platform.stratascratch.com/coding/9917-average-salaries/official-solution?code_type=3

**Difficulty:** Easy

**Problem ID:** 9917

## Problem Summary

Compare each employee's salary with the average salary of their department.

Return:

* Department
* First Name
* Salary
* Department Average Salary

---

## My Solution

```sql
WITH avg_salary AS
(
    SELECT
        department,
        AVG(salary) AS average_salary
    FROM employee
    GROUP BY department
)

SELECT
    e.department,
    e.first_name,
    e.salary,
    a.average_salary
FROM employee e
LEFT JOIN avg_salary a
    ON e.department = a.department;
```

---

## Official Solution

```sql
WITH dept_avg AS
(
    SELECT
        department,
        AVG(salary) AS avg_salary
    FROM employee
    GROUP BY department
)

SELECT
    e.department,
    e.first_name,
    e.salary,
    d.avg_salary
FROM employee e
JOIN dept_avg d
    ON e.department = d.department
ORDER BY e.department;
```

---

## My Thought Process

The problem asks for information at two different levels:

### Employee-Level Information

* Employee name
* Employee salary

### Department-Level Information

* Average salary within each department

Since the average salary does not exist in the original table, I first needed to calculate it.

I created a CTE that grouped employees by department and calculated:

```sql
AVG(salary)
```

This produced one row per department containing the department's average salary.

I then joined that result back to the employee table so that every employee record could be displayed alongside their department's average salary.

---

## What I Did Well

### Identified The Correct Granularity

One of the most important SQL skills is recognizing the level at which a metric should be calculated.

In this problem:

```text
Employee Salary → Employee Level
Department Average Salary → Department Level
```

I correctly recognized that the average salary needed to be calculated separately at the department level.

---

### Used The Aggregate → Join Back Pattern

This problem follows a very common analytics workflow:

```text
Aggregate Data
→ Join Back To Detailed Records
→ Compare Individuals To Their Group
```

My solution follows this pattern exactly.

---

### Kept The Logic Readable

By calculating department averages in a separate CTE, the query becomes easier to read and debug.

The CTE clearly answers:

```text
What is the average salary for each department?
```

before the final query answers:

```text
Which employees belong to each department?
```

---

## Difference Between My Solution And The Official Solution

The core logic is identical.

Both solutions:

1. Calculate average salary by department
2. Join the averages back to employee records
3. Display employee salary alongside department average salary

The main difference is the join type.

### My Solution

```sql
LEFT JOIN
```

### Official Solution

```sql
INNER JOIN
```

---

## Was My LEFT JOIN Wrong?

No.

Because the department averages were generated directly from the employee table, every employee department will have a matching record in the average salary CTE.

This means:

```text
LEFT JOIN and INNER JOIN produce the same result.
```

For this dataset, both queries return identical outputs.

---

## Why The Official Solution Used INNER JOIN

The official solution uses:

```sql
JOIN
```

which is equivalent to:

```sql
INNER JOIN
```

This communicates an assumption:

```text
Every employee should have a matching department average.
```

Since that assumption is guaranteed to be true, the INNER JOIN is slightly more precise.

---

## SQL Concept: Aggregate → Join Back

This is one of the most common SQL patterns used in analytics.

### Step 1: Create A Summary Table

```sql
SELECT
    department,
    AVG(salary)
FROM employee
GROUP BY department;
```

Example output:

| department | avg_salary |
| ---------- | ---------- |
| Audit      | 950        |
| Management | 175000     |
| Sales      | 65000      |

---

### Step 2: Join The Summary Back

```sql
employee
JOIN department_average
```

Example output:

| employee | salary | avg_salary |
| -------- | ------ | ---------- |
| Jason    | 1000   | 950        |
| Celine   | 1000   | 950        |
| Katty    | 150000 | 175000     |

This pattern appears constantly in real-world analytics.

Examples include:

* Employee vs Department Average
* Customer vs Regional Average
* Product vs Category Average
* Store vs National Average

---

## Biggest Takeaway

The most important lesson from this problem was not the SQL syntax itself.

It was recognizing that:

```text
Different metrics can exist at different levels of granularity.
```

When a problem mixes:

```text
Individual-level data
+
Group-level metrics
```

a common solution pattern is:

```text
GROUP BY
→ Calculate Aggregates
→ Join Back
```

This pattern appears frequently in analytics, business intelligence, and data science work.

---

## Key SQL Concepts Practiced

### AVG()

Calculating average salary.

```sql
AVG(salary)
```

---

### GROUP BY

Aggregating data at the department level.

```sql
GROUP BY department
```

---

### CTEs

Separating calculation logic from reporting logic.

```sql
WITH avg_salary AS (...)
```

---

### Joins

Combining department-level metrics with employee-level records.

```sql
JOIN avg_salary
ON employee.department = avg_salary.department
```

---

### Granularity

Understanding the difference between:

```text
Employee-Level Data
vs
Department-Level Data
```

and how to combine them correctly.

---

## Future Reminder

Whenever a problem asks me to compare:

```text
An individual
vs
Their group
```

I should immediately consider:

```text
GROUP BY
→ Aggregate
→ Join Back
```

because it is one of the most common SQL patterns used in analytics.
