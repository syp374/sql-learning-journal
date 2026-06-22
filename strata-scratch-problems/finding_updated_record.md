# Finding Updated Records

**Source:** StrataScratch

**Link:** https://platform.stratascratch.com/coding/10299-finding-updated-records?code_type=3

**Difficulty:** Easy

**Problem ID:** 10299

## Problem Summary

The employee salary table contains historical salary records.

Since salaries are assumed to be non-decreasing over time and no timestamps are available, the current salary for an employee is defined as:

```text
The maximum salary associated with that employee.
```

For each employee, return:

* id
* first_name
* last_name
* department_id
* current salary

If multiple records share the same maximum salary, any one of them may be returned.

Results should be ordered by employee ID.

---

## My Initial Approach

My first instinct was to aggregate salaries by employee.

```sql
SELECT
    id,
    first_name,
    last_name,
    department_id,
    MAX(salary) AS current_salary
FROM ms_employee_salary
GROUP BY id;
```

At first glance this seemed reasonable because the problem defines current salary as:

```text
Maximum salary per employee.
```

However, this approach contains a subtle flaw.

---

## The Problem I Discovered

One employee (Monica) had:

| id | department_id | salary |
| -- | ------------- | ------ |
| 1  | 100           | 70000  |
| 1  | 200           | 90000  |

My query correctly returned:

```text
90000
```

for the salary.

However, the department_id could come from:

```text
100
```

or

```text
200
```

depending on the database engine.

The issue is that:

```sql
MAX(salary)
```

identifies a value.

It does not identify the row that produced that value.

---

## Key Lesson

Aggregates return values.

They do not automatically preserve relationships to other columns.

Whenever I need:

```text
Maximum value
AND
Other columns from the same row
```

I need an additional step.

---

## My Solution

I created a CTE containing the maximum salary for each employee.

```sql
WITH maxlist AS
(
    SELECT
        id,
        MAX(salary) AS current_salary
    FROM ms_employee_salary
    GROUP BY id
)
```

Then I joined back to the original table.

```sql
SELECT
    x.id,
    m.first_name,
    m.last_name,
    m.department_id,
    x.current_salary
FROM maxlist x
LEFT JOIN ms_employee_salary m
    ON x.id = m.id
   AND x.current_salary = m.salary
ORDER BY x.id ASC;
```

This ensured that:

* The salary came from the correct row
* The department_id came from the same row as the maximum salary

---

## StrataScratch Solution

```sql
SELECT
    id,
    first_name,
    last_name,
    department_id,
    salary
FROM
(
    SELECT *,
           ROW_NUMBER() OVER
           (
               PARTITION BY id
               ORDER BY salary DESC,
                        department_id DESC
           ) AS rn
    FROM ms_employee_salary
) s
WHERE rn = 1
ORDER BY id ASC;
```

---

## Comparing My Solution vs StrataScratch Solution

### My Approach

Pattern:

```text
Aggregate
    ↓
Join Back
    ↓
Retrieve Matching Row
```

Advantages:

* Easy to understand
* Common SQL pattern
* Works across many SQL dialects

Disadvantages:

* Requires multiple steps
* Additional join

---

### StrataScratch Approach

Pattern:

```text
Rank Rows
    ↓
Select Top Row
```

Advantages:

* Shorter
* Cleaner
* Avoids join
* Easier to extend

Disadvantages:

* Requires familiarity with window functions

---

## What I Learned About Window Functions

The biggest lesson from this problem is that:

```text
ROW_NUMBER()
```

is not just for ranking.

It is often the best tool for selecting a specific row within a group.

Examples:

* Highest salary employee
* Most recent purchase
* Latest login
* First transaction
* Current employee record

When I see:

```text
Find the row with the highest...
Find the row with the latest...
Find the row with the largest...
```

I should consider:

```sql
ROW_NUMBER()
```

before immediately thinking about aggregation.

---

## Mental Models

### Aggregation

Use aggregation when I only need the value.

Examples:

```sql
MAX(salary)
MIN(price)
AVG(score)
SUM(revenue)
```

---

### Window Functions

Use window functions when I need the row.

Examples:

```text
Employee with highest salary
Most recent purchase
Latest event
Top-selling product
```

A useful question to ask:

> Do I need the value, or do I need the row?

If I need the row, a window function is often the better solution.

---

## Future Reminder

Whenever I see:

```text
Highest
Lowest
Latest
Earliest
First
Last
```

ask:

> Is this actually a ROW_NUMBER() problem disguised as an aggregation problem?
