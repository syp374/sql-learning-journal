# Finding User Purchases

**Source:** StrataScratch

**Link:** https://platform.stratascratch.com/coding/10322-finding-user-purchases/official-solution?code_type=3

**Difficulty:** Medium

**Problem ID:** 10322

## Problem Summary

Identify returning active users by finding users who made a **second purchase within 1 to 7 days after their first purchase**.

Requirements:

* Only consider the first and second purchase
* Ignore same-day purchases
* Return the list of qualifying `user_id`s

---

## My Initial Solution

```sql
WITH purchase_list AS
(
    SELECT
        user_id,
        created_at,
        ROW_NUMBER() OVER (
            PARTITION BY user_id
            ORDER BY created_at ASC
        ) AS purchase_num
    FROM amazon_transactions
)

SELECT first_purchase.user_id
FROM
(
    SELECT *
    FROM purchase_list
    WHERE purchase_num = 1
) AS first_purchase
INNER JOIN
(
    SELECT *
    FROM purchase_list
    WHERE purchase_num = 2
) AS second_purchase
    ON first_purchase.user_id = second_purchase.user_id
WHERE DATEDIFF(
        second_purchase.created_at,
        first_purchase.created_at
      ) BETWEEN 1 AND 7;
```

---

## My Thought Process

My initial idea was:

1. Rank each purchase chronologically for every user
2. Extract the first purchase
3. Extract the second purchase
4. Join them together
5. Calculate the date difference

This felt very natural because the question explicitly asks about:

```text
First Purchase
Second Purchase
```

Using `ROW_NUMBER()` seemed like the most direct way to identify those purchases.

---

## What I Did Well

### Correctly Identified The Core Logic

The key challenge was determining:

```text
What is a user's first purchase?
What is a user's second purchase?
```

Using:

```sql
ROW_NUMBER()
```

was a reasonable approach.

Once I had purchase numbers assigned, calculating the date difference became straightforward.

---

### Used Window Functions

Instead of relying on self-joins or nested aggregation, I used:

```sql
ROW_NUMBER()
```

to explicitly rank purchases.

This is often easier to read and maintain.

---

## The Edge Case I Missed

The biggest issue with my solution is:

```text
Multiple purchases on the same day
```

The prompt states:

> Ignore same-day purchases.

I initially interpreted this as:

```text
Exclude date differences equal to zero.
```

which led to:

```sql
DATEDIFF(...) BETWEEN 1 AND 7
```

However, the actual issue is more subtle.

---

## Example

Suppose a user makes:

| User | Purchase Date |
| ---- | ------------- |
| 100  | 2020-03-01    |
| 100  | 2020-03-01    |
| 100  | 2020-03-05    |

My ranking becomes:

| Purchase Date | Purchase Number |
| ------------- | --------------- |
| 2020-03-01    | 1               |
| 2020-03-01    | 2               |
| 2020-03-05    | 3               |

Then I compare:

```text
Purchase #1
vs
Purchase #2
```

which gives:

```text
0 days
```

The user would be excluded.

However, according to the business requirement:

```text
Same-day purchases should be ignored.
```

The correct comparison should be:

```text
2020-03-01
vs
2020-03-05
```

which is:

```text
4 days
```

and should qualify.

---

## Why The StrataScratch Solution Is More Robust

The official solution first removes duplicate purchase dates:

```sql
WITH daily AS
(
    SELECT DISTINCT
        user_id,
        DATE(created_at) AS purchase_date
    FROM amazon_transactions
)
```

This transforms:

| User | Purchase Date |
| ---- | ------------- |
| 100  | 2020-03-01    |
| 100  | 2020-03-01    |
| 100  | 2020-03-05    |

into:

| User | Purchase Date |
| ---- | ------------- |
| 100  | 2020-03-01    |
| 100  | 2020-03-05    |

Only then does it assign rankings.

This ensures:

```text
Purchase #1 = first unique purchase day
Purchase #2 = second unique purchase day
```

which matches the business requirement.

---

## Another Lesson: Read The Prompt Carefully

Initially I focused on:

```text
Second purchase
```

and immediately thought:

```sql
ROW_NUMBER() = 2
```

However, the phrase:

```text
Ignore same-day purchases
```

fundamentally changes what:

```text
Second purchase
```

actually means.

It really means:

```text
Second unique purchase day
```

not:

```text
Second transaction record
```

This distinction is easy to miss.

---

## Comparing My Solution To The Official Solution

### My Solution

**Advantages**

* Simple
* Easy to understand
* Direct use of ROW_NUMBER()
* Efficient for datasets without same-day purchases

**Disadvantages**

* Fails when users have multiple purchases on the same day
* Treats transactions as purchases rather than purchase days

---

### Official Solution

**Advantages**

* Handles same-day purchases correctly
* Matches the business requirement exactly
* More robust against edge cases

**Disadvantages**

* Slightly more verbose
* Requires an additional preprocessing step

---

## Key SQL Concepts Practiced

### ROW_NUMBER()

Used to rank purchases chronologically.

```sql
ROW_NUMBER()
OVER (
    PARTITION BY user_id
    ORDER BY created_at
)
```

---

### DISTINCT

Removing duplicate purchase dates before ranking.

```sql
SELECT DISTINCT
    user_id,
    purchase_date
```

---

### DATEDIFF()

Calculating elapsed days between purchases.

```sql
DATEDIFF(second_date, first_date)
```

---

### Business Logic vs Technical Logic

A query can be technically correct while still violating the intended business definition.

This problem reinforced the importance of understanding:

```text
What exactly is being counted?
```

before writing SQL.

---

## Biggest Takeaway

My solution worked under the assumption:

```text
Second purchase = second transaction
```

The actual requirement was:

```text
Second purchase = second unique purchase day
```

The difference is subtle, but it completely changes the behavior of the query for users with multiple purchases on the same day.

This was a valuable reminder that interview SQL problems are often less about syntax and more about correctly interpreting business requirements.
