# Users By Average Session Time

**Source:** StrataScratch

**Link:** https://platform.stratascratch.com/coding/10352-users-by-avg-session-time?code_type=3

**Difficulty:** Medium

**Problem ID:** 10352

## Problem Summary

Calculate each user's average session time.

A session is defined as the time difference between:

* The latest `page_load` event of the day
* The earliest `page_exit` event of the day

Additional requirements:

* Assume each user has only one session per day
* If multiple page loads occur, use the latest one
* If multiple page exits occur, use the earliest one
* Only consider sessions where the page load occurs before the page exit
* Return each user's average session time

---

## My Initial Thought Process

My first instinct was to solve the problem in three stages:

1. Identify the latest page load for each user-day
2. Identify the earliest page exit for each user-day
3. Join the two datasets and calculate session durations

Since the problem required selecting specific records within each user-day group, I immediately thought of using window functions and `ROW_NUMBER()`.

---

## My Solution

```sql
WITH load_list AS
(
    SELECT *
    FROM
    (
        SELECT *,
               DATE(`timestamp`) AS `date`,
               ROW_NUMBER() OVER
               (
                   PARTITION BY user_id, DATE(`timestamp`)
                   ORDER BY `timestamp` DESC
               ) AS timern
        FROM facebook_web_log
        WHERE `action` = 'page_load'
    ) AS load_rn
    WHERE timern = 1
),

exit_list AS
(
    SELECT *
    FROM
    (
        SELECT *,
               DATE(`timestamp`) AS `date`,
               ROW_NUMBER() OVER
               (
                   PARTITION BY user_id, DATE(`timestamp`)
                   ORDER BY `timestamp` ASC
               ) AS timern
        FROM facebook_web_log
        WHERE `action` = 'page_exit'
    ) AS load_rn
    WHERE timern = 1
)

SELECT
    l.user_id,
    AVG(e.timestamp - l.timestamp) AS avg_session_time
FROM load_list l
JOIN exit_list e
    ON l.user_id = e.user_id
   AND l.date = e.date
WHERE l.timestamp < e.timestamp
GROUP BY l.user_id;
```

---

## The Bug That Tricked Me

The query executed successfully.

It returned:

```text
user_id    avg_session_time
0          5072.5
1          35
```

At first, I assumed my logic was mostly correct because:

* No syntax errors occurred
* The query produced output
* The values looked somewhat reasonable

However, the answer was incorrect.

This was a great reminder that:

> A query returning results does not mean the query is correct.

---

## Mistake #1: Incorrect DATETIME Arithmetic

The issue was here:

```sql
e.timestamp - l.timestamp
```

I assumed subtracting two timestamps would automatically return a time difference.

It does not.

In MySQL, datetime subtraction performs implicit type conversion and treats the datetime values as numeric values.

For example:

```sql
'2020-01-01 10:00:00'
```

is interpreted roughly as:

```text
20200101100000
```

and:

```sql
'2020-01-01 09:00:00'
```

becomes:

```text
20200101090000
```

The subtraction becomes:

```text
20200101100000
-
20200101090000
=
10000
```

which is not:

```text
3600 seconds
```

or:

```text
60 minutes
```

The query was producing mathematically valid numbers that had no meaningful interpretation as session durations.

---

## Key Lesson: Use Explicit Time Functions

The correct approach is:

```sql
TIMESTAMPDIFF(SECOND, start_time, end_time)
```

This tells MySQL exactly how to compute the difference.

Examples:

```sql
TIMESTAMPDIFF(SECOND, start_time, end_time)
```

```sql
TIMESTAMPDIFF(MINUTE, start_time, end_time)
```

```sql
TIMESTAMPDIFF(HOUR, start_time, end_time)
```

Whenever working with dates and times, explicit date functions are safer than relying on arithmetic operators.

---

## StrataScratch Solution

```sql
WITH loads AS
(
  SELECT
      user_id,
      DATE(timestamp) AS day,
      MAX(timestamp) AS load_time
  FROM facebook_web_log
  WHERE action = 'page_load'
  GROUP BY user_id, DATE(timestamp)
),

exits AS
(
  SELECT
      user_id,
      DATE(timestamp) AS day,
      MIN(timestamp) AS exit_time
  FROM facebook_web_log
  WHERE action = 'page_exit'
  GROUP BY user_id, DATE(timestamp)
),

sessions AS
(
  SELECT
      l.user_id,
      l.load_time,
      e.exit_time,
      TIMESTAMPDIFF(
          SECOND,
          l.load_time,
          e.exit_time
      ) AS session_duration
  FROM loads l
  JOIN exits e
      ON l.user_id = e.user_id
     AND l.day = e.day
  WHERE l.load_time < e.exit_time
)

SELECT
    user_id,
    AVG(session_duration) AS avg_session_duration
FROM sessions
GROUP BY user_id;
```

---

## Comparing My Solution vs The Suggested Solution

### What I Did Well

I correctly identified:

* The latest page load
* The earliest page exit
* The need to compare events within each user-day
* The need to filter invalid sessions

My overall logic was correct.

---

### Where My Solution Was More Complicated

I used:

```sql
ROW_NUMBER()
```

to identify:

* Latest load
* Earliest exit

However, the problem only required:

```sql
MAX(timestamp)
```

for loads and

```sql
MIN(timestamp)
```

for exits.

The StrataScratch solution recognized that this is fundamentally an aggregation problem rather than a ranking problem.

---

### Window Functions vs Aggregations

My approach:

```sql
ROW_NUMBER()
```

StrataScratch approach:

```sql
MAX()
MIN()
```

Both can solve the problem.

However:

```sql
MAX()
MIN()
```

are:

* Simpler
* Easier to read
* More computationally efficient

This was a valuable lesson in choosing the simplest tool that solves the problem.

---

## Key SQL Concepts Practiced

### Window Functions

Using:

```sql
ROW_NUMBER()
```

to select specific rows within groups.

---

### Aggregations

Using:

```sql
MAX()
MIN()
AVG()
```

to summarize data.

---

### Datetime Functions

Using:

```sql
TIMESTAMPDIFF()
```

to correctly calculate elapsed time.

---

### Query Design

A problem can often be solved multiple ways.

The best solution is not necessarily the first correct solution.

It is often the simplest and most maintainable one.

---

## Mental Models

### Window Functions

Use window functions when you need to identify rows.

Examples:

* First purchase
* Latest login
* Top product per category

---

### Aggregations

Use aggregate functions when you only need values.

Examples:

* Earliest timestamp
* Latest timestamp
* Maximum revenue
* Average score

If all you need is the value itself, consider `MIN()` or `MAX()` before reaching for `ROW_NUMBER()`.

---

## Future Reminder

Before using a window function, I should ask if I actually need a row or just a value.

And whenever working with dates and timestamps:

> Use explicit datetime functions. Never assume subtraction will produce a meaningful time interval.
