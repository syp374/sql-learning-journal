# Acceptance Rate By Date

**Source:** StrataScratch

**Link:** https://platform.stratascratch.com/coding/10285-acceptance-rate-by-date?code_type=3

**Difficulty:** Medium

**Problem ID:** 10285

## Problem Summary

Calculate the friend acceptance rate for each date when friend requests were sent.

Definitions:

* A request is created when `action = 'sent'`
* A request is accepted when `action = 'accepted'`
* Acceptance may occur on any later date
* Some requests are never accepted

Acceptance Rate:

```text
(Number of accepted requests)
/
(Number of sent requests)
```

The output should only include dates where:

* Requests were sent
* At least one of those requests was eventually accepted

---

## My Initial Approach

I separated the table into two CTEs:

```sql
WITH table_sent AS
(
    SELECT *
    FROM fb_friend_requests
    WHERE action = 'sent'
),

table_accepted AS
(
    SELECT *
    FROM fb_friend_requests
    WHERE action = 'accepted'
)

SELECT
    s.date,
    COUNT(a.user_id_sender) / COUNT(s.user_id_sender) AS acceptance_rate
FROM table_sent s
LEFT JOIN table_accepted a
    ON s.user_id_sender = a.user_id_sender
   AND s.user_id_receiver = a.user_id_receiver
GROUP BY s.date;
```

This produced the correct answer.

At first, I felt reasonably confident about the solution.

However, I became confused by one specific requirement.

---

## The Requirement That Confused Me

The problem states:

> The output will only include dates where requests were sent and at least one of them was accepted.

I started wondering:

```text
How can I know whether a request was accepted
if the acceptance might happen on a completely different date?
```

For a while, I felt like my solution was not actually satisfying the requirement.

---

## The Mistake In My Reasoning

I was mentally treating the accepted table as:

```text
A table of acceptance events by date
```

which led me to think:

```text
If I group by sent date,
how can I count acceptances that happened later?
```

This made the logic feel much more complicated than it really was.

---

## The Key Realization

The join changes the meaning of the accepted table.

Before the join:

```text
Accepted table = acceptance events
```

After the join:

```text
Accepted table = acceptance indicator
```

The join:

```sql
ON s.user_id_sender = a.user_id_sender
AND s.user_id_receiver = a.user_id_receiver
```

is not asking:

```text
When was the request accepted?
```

It is asking:

```text
Was the request ever accepted?
```

That distinction completely changed how I thought about the problem.

---

## Visual Example

### Sent Requests

| Sender | Receiver | Sent Date |
| ------ | -------- | --------- |
| A      | B        | Jan 1     |
| C      | D        | Jan 1     |
| E      | F        | Jan 2     |

### Accepted Requests

| Sender | Receiver | Accepted Date |
| ------ | -------- | ------------- |
| A      | B        | Jan 10        |

After the join:

| Sent Date | Request | Accepted? |
| --------- | ------- | --------- |
| Jan 1     | A → B   | Yes       |
| Jan 1     | C → D   | No        |
| Jan 2     | E → F   | No        |

Notice that:

```text
Jan 10 completely disappears from the calculation.
```

The only thing that matters is:

```text
Did a matching accepted record exist?
```

---

## Understanding COUNT After a LEFT JOIN

Because of the LEFT JOIN:

```sql
COUNT(a.user_id_sender)
```

counts only rows where:

```text
A matching accepted request exists
```

For the example above:

### Jan 1

```text
Accepted Requests = 1
Sent Requests = 2

Acceptance Rate = 1 / 2
```

### Jan 2

```text
Accepted Requests = 0
Sent Requests = 1

Acceptance Rate = 0 / 1
```

---

## The Requirement I Initially Missed

My original query would also return:

```text
Jan 2 → 0
```

However, the problem specifies:

> Only include dates where at least one request was accepted.

To explicitly enforce that requirement:

```sql
HAVING COUNT(a.user_id_sender) > 0
```

can be added after grouping.

```sql
SELECT
    s.date,
    COUNT(a.user_id_sender) * 1.0
    / COUNT(s.user_id_sender) AS acceptance_rate
FROM table_sent s
LEFT JOIN table_accepted a
    ON s.user_id_sender = a.user_id_sender
   AND s.user_id_receiver = a.user_id_receiver
GROUP BY s.date
HAVING COUNT(a.user_id_sender) > 0;
```

---

## Comparing My Solution To StrataScratch

### My Solution

```sql
LEFT JOIN accepted
```

### StrataScratch Solution

```sql
RIGHT JOIN sent
```

Conceptually, these are doing the same thing.

Both preserve:

```text
Sent Requests
```

and check whether:

```text
A matching acceptance exists.
```

The core logic is identical.

---

## Biggest Lesson From This Problem

The biggest lesson was not about aggregation.

It was about understanding what a JOIN represents.

I initially thought:

```text
Accepted table = acceptance dates
```

But after the join:

```text
Accepted table = acceptance flag
```

The acceptance date itself becomes irrelevant.

The joined table is simply helping answer:

```text
Was this request eventually accepted?
```

---

## Key SQL Concepts Practiced

### Common Table Expressions (CTEs)

Using separate tables for:

```sql
sent
accepted
```

to simplify logic.

---

### LEFT JOIN

Using a LEFT JOIN to preserve all requests that were sent.

---

### Counting Matches After A Join

Using:

```sql
COUNT(column)
```

to count only non-null matches from the joined table.

---

### HAVING

Filtering grouped results based on aggregate conditions.

---

## Mental Models

### Before The Join

```text
Accepted table = events
```

### After The Join

```text
Accepted table = yes/no lookup
```

Whenever a join is performed, ask:

> What information is this table contributing to the final result?

Understanding the answer is often more important than understanding the syntax.

---

## Future Reminder

When solving SQL problems involving joins:

Do not only think about:

```text
How do I join these tables?
```

Also think about:

```text
What question does the join allow me to answer?
```

In this problem, the join was not answering:

```text
When was the request accepted?
```

It was answering:

```text
Was the request ever accepted?
```

Recognizing that distinction made the entire problem much easier to understand.
