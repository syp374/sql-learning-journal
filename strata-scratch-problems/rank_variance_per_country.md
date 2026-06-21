# Rank Variance Per Country

**Source:** StrataScratch

**Link:** https://platform.stratascratch.com/coding/2007-rank-variance-per-country?code_type=3

**Difficulty:** Hard

**Problem ID:** 2007

## Problem Summary

Compare the total number of comments made by users in each country during December 2019 and January 2020.

For each month:

* Calculate total comments by country
* Rank countries in descending order of total comments
* Countries with the same total comments should share the same rank (`DENSE_RANK`)
* Return countries whose rank improved from December to January (smaller rank number)

---

## My Approach

I broke the problem into three logical steps:

### Step 1: Aggregate Comment Counts

First, I joined the user table and comment table and calculated total comments by:

* Month
* Country

### Step 2: Rank Countries Within Each Month

After aggregation, I used `DENSE_RANK()` to rank countries by total comments.

### Step 3: Compare December vs January Rankings

I created a ranked dataset and then:

* Extracted December rankings
* Extracted January rankings
* Joined both datasets on country
* Returned countries whose January rank was better than their December rank

---

## Final Solution

```sql
WITH rank_list AS
(
    SELECT
        MONTH(created_at) AS `month`,
        country,
        SUM(number_of_comments) AS total_comments,
        DENSE_RANK() OVER
        (
            PARTITION BY MONTH(created_at)
            ORDER BY SUM(number_of_comments) DESC
        ) AS dr
    FROM fb_active_users u
    LEFT JOIN fb_comments_count c
        ON u.user_id = c.user_id
    WHERE created_at BETWEEN '2019-12-01' AND '2020-01-31'
    GROUP BY `month`, country
)

SELECT jan.country
FROM
(
    SELECT *
    FROM rank_list
    WHERE `month` = 12
) AS `dec`
INNER JOIN
(
    SELECT *
    FROM rank_list
    WHERE `month` = 1
) AS jan
ON `dec`.country = jan.country
WHERE `dec`.dr > jan.dr;
```

---

## Mistakes & Debugging Notes

### 1. Confusing `GROUP BY` with `PARTITION BY`

My initial ranking query looked something like:

```sql
DENSE_RANK() OVER (
    PARTITION BY month(created_at), country
    ORDER BY SUM(number_of_comments) DESC
)
```

This produced:

```text
1
1
1
1
1
...
```

for every rank.

At first I thought the ranking function was broken.

The actual issue was that each `(month, country)` combination was already unique after aggregation.

As a result:

```text
December + USA
December + China
December + Brazil
```

all became separate partitions.

Each partition contained only one row, and a partition with one row will always receive rank 1.

### Key Lesson

`PARTITION BY` defines who competes against whom.

Since countries should compete against other countries within the same month, the correct partition is:

```sql
PARTITION BY month(created_at)
```

---

### 2. Better Understanding of Window Functions

A major takeaway from this problem was understanding the difference between:

```sql
GROUP BY
```

and

```sql
PARTITION BY
```

#### GROUP BY

Creates aggregation buckets.

Example:

```text
USA:
    5 comments
    3 comments

Total = 8
```

Rows are collapsed.

#### PARTITION BY

Creates comparison groups for window functions.

Rows remain visible.

Window functions can look across rows within the partition.

### Mental Model

```text
Raw Data
    ↓
GROUP BY
    ↓
Aggregated Data
    ↓
PARTITION BY
    ↓
Ranking / Window Calculations
```

---

### 3. Reserved Keyword Alias Issue

I initially used:

```sql
AS dec
```

for my December table alias.

This caused a syntax error.

The reason is that `DEC` is a reserved SQL keyword related to the `DECIMAL` data type.

Changing the alias to:

```sql
AS `dec`
```

resolved the issue.

### Key Lesson

When a syntax error appears near a table alias, check whether the alias is a reserved SQL keyword.

---

### 4. CASE WHEN Was Unnecessary

My first instinct was to compare rankings using:

```sql
CASE WHEN jan_rank < dec_rank THEN ...
```

However, the problem only required returning countries whose rankings improved.

A simple filter was sufficient:

```sql
WHERE dec_rank > jan_rank
```

This simplified the solution considerably.

---

## Key SQL Concepts Practiced

* Joins
* Aggregations
* Window Functions
* DENSE_RANK()
* Common Table Expressions (CTEs)
* Self-Comparison Across Time Periods
* Ranking Logic
* SQL Debugging

---

## Future Reminder

Whenever a window function produces unexpected results, always ask:

**"What rows are actually inside each partition?"**

Many ranking issues come from incorrectly defining the partition rather than the ranking function itself.
