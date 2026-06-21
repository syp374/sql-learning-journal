# Monthly Percentage Difference

**Source:** StrataScratch (https://platform.stratascratch.com/coding/10319-monthly-percentage-difference?code_type=3)
**Difficulty:** Hard
**Problem ID:** 10319

## Problem Summary

Given a table of purchases by date, calculate the month-over-month percentage change in revenue.

The output should include:

* Year-month (`YYYY-MM`)
* Percentage change in revenue compared to the previous month
* Results rounded to 2 decimal places
* Sorted chronologically

Percentage change is calculated as:

```text
((Current Month Revenue - Previous Month Revenue)
 / Previous Month Revenue) * 100
```

---

## My Approach

I broke the problem into two steps.

### Step 1: Calculate Monthly Revenue

First, I aggregated transaction values by month using:

```sql
SUM(value)
```

and formatted the date as:

```sql
DATE_FORMAT(created_at, '%Y-%m')
```

to create monthly revenue totals.

### Step 2: Retrieve Previous Month Revenue

To calculate month-over-month growth, I needed access to the previous month's revenue.

This is a perfect use case for the window function:

```sql
LAG()
```

which allows us to access values from a previous row without performing a self join.

I used `LAG()` to create a column containing the previous month's revenue and then applied the percentage change formula.

---

## My Solution

```sql
WITH rev_list AS
(
    SELECT
        DATE_FORMAT(created_at,'%Y-%m') AS `date`,
        SUM(value) AS revenue,
        LAG(SUM(value)) OVER () AS rev_last_month
    FROM sf_transactions
    GROUP BY DATE_FORMAT(created_at,'%Y-%m')
)

SELECT
    `date`,
    ROUND(
        ((revenue - rev_last_month) / rev_last_month) * 100,
        2
    ) AS perc_change_rev
FROM rev_list
ORDER BY `date`;
```

---

## Recommended Solution

```sql
SELECT
    DATE_FORMAT(created_at,'%Y-%m') AS ym,
    ROUND(
        (
            SUM(value)
            - LAG(SUM(value)) OVER ()
        )
        /
        LAG(SUM(value)) OVER ()
        * 100,
        2
    ) AS revenue_diff_pct
FROM sf_transactions
GROUP BY ym
ORDER BY ym;
```

---

## Comparing My Solution vs Recommended Solution

Both solutions produce the same result.

The main difference is readability and maintainability.

### My Approach

I used a CTE to separate the logic into stages:

1. Calculate monthly revenue
2. Calculate previous month's revenue
3. Calculate percentage difference

This made the query easier to understand and debug.

### Recommended Approach

The recommended solution performs everything in a single query by applying `LAG()` directly to the aggregated revenue values.

This results in a shorter query but is slightly harder to read at first glance.

### Key Lesson

There is often a trade-off between:

* Conciseness
* Readability

In analytics work, readability is frequently more important than writing the shortest query possible.

---

## Key SQL Concepts Practiced

### 1. Window Functions

This problem reinforced how window functions allow calculations across rows without collapsing the dataset.

Unlike aggregate functions, window functions preserve row-level visibility.

---

### 2. LAG()

`LAG()` retrieves data from a previous row.

Example:

| Month | Revenue |
| ----- | ------- |
| Jan   | 100     |
| Feb   | 150     |
| Mar   | 120     |

Applying:

```sql
LAG(revenue)
```

produces:

| Month | Revenue | Previous Revenue |
| ----- | ------- | ---------------- |
| Jan   | 100     | NULL             |
| Feb   | 150     | 100              |
| Mar   | 120     | 150              |

This makes month-over-month calculations straightforward.

---

### 3. Window Functions Can Operate on Aggregated Results

One concept I found interesting was:

```sql
LAG(SUM(value))
```

At first glance this feels strange because:

```sql
SUM()
```

is an aggregate function and

```sql
LAG()
```

is a window function.

However, SQL logically performs:

```text
GROUP BY
    ↓
SUM(value)
    ↓
Window Function
```

This means the window function operates on the aggregated monthly revenue values.

### Mental Model

```text
Raw Transactions
    ↓
GROUP BY Month
    ↓
Monthly Revenue
    ↓
LAG()
    ↓
Previous Month Revenue
```

---

## Alternative Solution

A common alternative would be a self join.

Example conceptually:

```text
Current Month Revenue
    JOIN
Previous Month Revenue
```

However, this approach is more verbose and less elegant than using:

```sql
LAG()
```

Window functions are generally the preferred solution for sequential time-series comparisons.

---

## Key Takeaways

### LAG() is often the first tool to consider when:

* Comparing consecutive months
* Comparing consecutive days
* Tracking growth rates
* Measuring changes over time

---

### Window Functions Reduce the Need for Self Joins

Many historical comparison problems that once required complex self joins can be solved more cleanly with:

* LAG()
* LEAD()
* ROW_NUMBER()
* RANK()
* DENSE_RANK()

---

### Readability Matters

My solution used an extra CTE but was easier to reason about.

For learning, debugging, and collaboration, a slightly longer query is often preferable to a highly compressed one.

---

## Future Reminder

Whenever a problem asks for:

* Previous period
* Previous month
* Previous transaction
* Previous observation

my first instinct should be:

**"Can this be solved using LAG() or LEAD() window functions?"**
