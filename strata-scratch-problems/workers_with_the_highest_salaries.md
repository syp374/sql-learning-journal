# Workers With The Highest Salaries

**Source:** StrataScratch

**Link:** https://platform.stratascratch.com/coding/10353-workers-with-the-highest-salaries/official-solution?code_type=3

**Difficulty:** Easy

**Problem ID:** 10353

## Problem Summary

Management wants to analyze only employees with official job titles.

Find the title(s) of the worker(s) with the highest salary among workers who have a corresponding record in the `title` table.

Requirements:

* Only include workers that exist in both tables
* Return all titles tied for the highest salary
* Output the job title(s)

---

## My Solution

```sql
WITH rank_list AS
(
    SELECT
        w.worker_id,
        w.salary,
        t.worker_title,
        DENSE_RANK() OVER (
            ORDER BY salary DESC
        ) AS rn
    FROM worker w
    INNER JOIN title t
        ON w.worker_id = t.worker_ref_id
)

SELECT worker_title AS highest_salary_titles
FROM rank_list
WHERE rn = 1;
```

---

## Official Solution

```sql
SELECT b.worker_title AS best_paid_title
FROM worker a
JOIN title b
    ON a.worker_id = b.worker_ref_id
WHERE a.salary =
(
    SELECT MAX(w.salary)
    FROM worker w
    JOIN title t
        ON w.worker_id = t.worker_ref_id
)
ORDER BY best_paid_title;
```

---

## My Thought Process

The phrase:

```text
highest salary
```

immediately made me think about ranking.

I joined the tables and assigned:

```sql
DENSE_RANK()
```

based on salary in descending order.

Then I simply selected:

```sql
WHERE rn = 1
```

to return all workers tied for the highest salary.

Because I used:

```sql
DENSE_RANK()
```

multiple workers sharing the same highest salary would automatically be included.

---

## What I Did Well

### Correctly Joined The Tables

The prompt specifically says:

> workers who have a corresponding record in the title table

Using:

```sql
INNER JOIN
```

ensured that only workers with valid title records were considered.

---

### Properly Handled Salary Ties

Using:

```sql
DENSE_RANK()
```

ensured that:

```text
Highest Salary = Rank 1
```

and all tied workers would be included.

For example:

| Worker | Salary |
| ------ | ------ |
| A      | 100000 |
| B      | 100000 |
| C      | 90000  |

Results:

| Worker | Rank |
| ------ | ---- |
| A      | 1    |
| B      | 1    |
| C      | 2    |

Filtering on:

```sql
WHERE rn = 1
```

returns both A and B.

---

## What I Could Have Done Better

The problem only asks for:

```text
Highest Salary
```

It does not ask for:

```text
Ranking all workers
```

Because of that, a simpler approach exists.

The official solution calculates:

```sql
MAX(salary)
```

and then returns workers whose salary matches that value.

Conceptually:

```text
Find maximum salary
→ Return matching titles
```

instead of:

```text
Rank everybody
→ Keep rank 1
```

---

## Comparing The Two Approaches

### My Approach

```sql
DENSE_RANK()
```

Advantages:

* Easy to understand
* Automatically handles ties
* Useful if rankings are needed later

Disadvantages:

* Calculates rankings for every worker
* More work than required
* Slightly over-engineered for this problem

---

### Official Approach

```sql
MAX(salary)
```

Advantages:

* Simpler
* More direct
* Expresses business logic clearly

Disadvantages:

* Requires a subquery

---

## SQL Concept: Don't Reach For Window Functions Too Quickly

One lesson from this problem is that:

```text
Not every highest-value problem requires ranking.
```

When I first learned window functions, I found myself using:

```sql
ROW_NUMBER()
RANK()
DENSE_RANK()
```

very frequently.

However, before using a ranking function, it is worth asking:

```text
Do I actually need a ranking?
```

Sometimes:

```sql
MAX()
MIN()
```

can solve the problem more directly.

---

## When Would DENSE_RANK Be Better?

Suppose the problem asked:

```text
Return the top 3 salaries.
```

or

```text
Return the 2nd highest salary.
```

Then:

```sql
DENSE_RANK()
```

would become a natural solution.

For example:

```sql
DENSE_RANK() OVER (
    ORDER BY salary DESC
)
```

allows easy filtering such as:

```sql
WHERE rn <= 3
```

or

```sql
WHERE rn = 2
```

---

## Biggest Takeaway

My solution was correct and handled ties properly.

However, it solved a:

```text
MAX()
```

problem using a:

```text
DENSE_RANK()
```

approach.

The lesson is:

> Before using a window function, ask whether a simpler aggregation function can solve the problem.

Sometimes the best SQL solution is not the most advanced one.

It is the one that most directly expresses the business question.

---

## Key SQL Concepts Practiced

### INNER JOIN

Keeping only workers with valid title records.

```sql
INNER JOIN title
ON worker.worker_id = title.worker_ref_id
```

---

### DENSE_RANK()

Assigning rankings while preserving ties.

```sql
DENSE_RANK() OVER (
    ORDER BY salary DESC
)
```

---

### MAX()

Finding the highest value in a dataset.

```sql
MAX(salary)
```

---

### Choosing The Right Tool

Not every problem that involves "highest" or "lowest" values requires a ranking function.

Sometimes aggregation is the simpler and more efficient solution.

---

## Future Reminder

Before writing a window function, ask:

```text
Do I need rankings?
Or do I only need the maximum/minimum value?
```

If the answer is:

```text
Just the maximum or minimum
```

then:

```sql
MAX()
MIN()
```

may be the cleaner solution.
