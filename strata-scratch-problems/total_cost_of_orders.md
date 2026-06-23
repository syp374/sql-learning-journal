# Total Cost Of Orders

**Source:** StrataScratch  

**Link:** https://platform.stratascratch.com/coding/10183-total-cost-of-orders/official-solution?code_type=3

**Difficulty:** Easy  

**Problem ID:** 10183

## Problem Summary

Find the total cost of each customer's orders.

Return:

- Customer ID
- Customer first name
- Total order cost

Sort the results alphabetically by first name.

---

## My Solution

```sql
SELECT
    c.id,
    c.first_name,
    SUM(o.total_order_cost) AS total_order_cost
FROM customers c
INNER JOIN orders o
    ON c.id = o.cust_id
GROUP BY c.id
ORDER BY c.first_name ASC;
```

---

## Official Solution

```sql
SELECT
    customers.id,
    customers.first_name,
    SUM(total_order_cost)
FROM orders
JOIN customers
    ON customers.id = orders.cust_id
GROUP BY
    customers.id,
    customers.first_name
ORDER BY customers.first_name ASC;
```

---

## My Thought Process

My first instinct was to use a:

```sql
LEFT JOIN
```

because the question asked for:

> total cost of each customer's orders

and did not explicitly state whether customers without orders should be included.

Initially I wrote something conceptually similar to:

```sql
FROM customers c
LEFT JOIN orders o
    ON c.id = o.cust_id
```

However, after looking at the expected output, I noticed that customers without purchases were not included.

That led me to switch to:

```sql
INNER JOIN
```

which only keeps customers that have matching order records.

---

## What I Did Well

### Correctly Identified The Relationship

The problem requires combining information from:

- `customers`
- `orders`

The join condition was straightforward:

```sql
c.id = o.cust_id
```

which links each order to its customer.

---

### Correct Use Of Aggregation

Because customers can have multiple orders, I needed:

```sql
SUM(total_order_cost)
```

to calculate the total amount spent.

---

### Recognized Join Type Matters

Even though the prompt did not explicitly mention excluding customers without purchases, I paid attention to the expected output and adjusted the join accordingly.

This is an important habit in SQL interviews because:

```text
Expected output often reveals hidden business rules.
```

---

## Mistake In My Solution

My query was:

```sql
GROUP BY c.id
```

while selecting:

```sql
c.first_name
```

at the same time.

```sql
SELECT
    c.id,
    c.first_name,
    SUM(...)
GROUP BY c.id
```

This works in MySQL because MySQL allows selecting non-grouped columns when they are functionally dependent on the grouped column.

Since:

```text
id uniquely identifies first_name
```

MySQL accepts the query.

---

## Why The Official Solution Is Better

The official solution uses:

```sql
GROUP BY
    customers.id,
    customers.first_name
```

which is more explicit and portable.

Advantages:

- Works across more SQL dialects
- Makes the grouping logic clearer
- Avoids relying on MySQL-specific behavior

---

## SQL Concept: Functional Dependency

This problem reminded me of an important SQL concept.

Suppose:

| id | first_name |
|----|------------|
| 1 | John |
| 2 | Sarah |

Each `id` maps to exactly one `first_name`.

Therefore:

```text
first_name is functionally dependent on id
```

Grouping by:

```sql
GROUP BY id
```

is enough for MySQL to determine the corresponding first name.

However, not all SQL engines allow this.

For example:

- PostgreSQL would reject this query.
- SQL Server would reject this query.
- MySQL allows it.

Because of this, it is generally safer to group by every non-aggregated column being selected.

---

## Join Type Lesson

### INNER JOIN

```sql
FROM customers c
INNER JOIN orders o
    ON c.id = o.cust_id
```

Returns:

```text
Only customers who have orders.
```

---

### LEFT JOIN

```sql
FROM customers c
LEFT JOIN orders o
    ON c.id = o.cust_id
```

Returns:

```text
All customers,
including customers with no orders.
```

Those customers would have:

```text
NULL order values.
```

---

## Biggest Takeaway

The most valuable lesson from this problem was not aggregation—it was understanding how join types affect the result set.

I initially assumed:

```text
All customers should appear.
```

But the expected output implied:

```text
Only customers with purchases should appear.
```

This reinforced an important habit:

> Always compare your assumptions against the expected output. Hidden business rules are often revealed there.

---

## Key SQL Concepts Practiced

### INNER JOIN

Combining customers with their orders.

```sql
INNER JOIN orders
ON customers.id = orders.cust_id
```

---

### Aggregation

Calculating total spending per customer.

```sql
SUM(total_order_cost)
```

---

### GROUP BY

Grouping rows at the customer level.

```sql
GROUP BY customer_id
```

---

### Functional Dependency

Understanding why MySQL allowed:

```sql
GROUP BY id
```

while still selecting:

```sql
first_name
```

---

## Future Reminder

When writing aggregation queries:

Ask yourself:

```text
What should happen to rows with no matching records?
```

The answer often determines whether you should use:

- INNER JOIN
- LEFT JOIN
- RIGHT JOIN

before writing any aggregation logic.
