# Counting Instances in Text

**Source:** StrataScratch

**Link:** https://platform.stratascratch.com/coding/9814-counting-instances-in-text/official-solution?code_type=3

**Difficulty:** Hard

**Problem ID:** 9814

## Problem Summary

Find the total number of times the exact words:

* bull
* bear

appear in the `contents` column.

Requirements:

* Count all occurrences
* Count multiple occurrences within the same row
* Matches should be case-insensitive
* Only count exact words
* Exclude substrings such as:

```text
bullish
bearing
bulls
```

Output:

| word | nentry |
| ---- | ------ |
| bull | count  |
| bear | count  |

---

## My Initial Approach

My first instinct was to identify rows containing the target words.

```sql
SELECT
    SUM(bull) AS bull,
    SUM(bear) AS bear
FROM
(
    SELECT *,
           CASE
               WHEN contents LIKE '%bull%'
               THEN COUNT(contents)
               ELSE 0
           END AS bull,
           CASE
               WHEN contents LIKE '%bear%'
               THEN COUNT(contents)
               ELSE 0
           END AS bear
    FROM google_file_store
    WHERE contents LIKE '%bull%'
       OR contents LIKE '%bear%'
    GROUP BY filename
) AS count_list;
```

This produced:

| bull | bear |
| ---- | ---- |
| 3    | 2    |

which matched the expected counts.

At first glance, this seemed correct.

---

## Mistake #1: Counting Rows Instead of Occurrences

After rereading the problem, I realized I was counting:

```text
Rows containing the word
```

instead of:

```text
Actual occurrences of the word
```

For example:

```text
bull bull bull
```

should contribute:

```text
3
```

occurrences.

My query would count:

```text
1
```

because the row contains the word.

This is a very different problem.

---

## Mistake #2: Exact Word Matching

I also noticed:

```sql
LIKE '%bull%'
```

would incorrectly count:

```text
bullish
```

and:

```sql
LIKE '%bear%'
```

would incorrectly count:

```text
bearing
```

The problem specifically requires:

```text
Exact words only
```

which means substring matching is insufficient.

---

## Mistake #3: Focusing on the Output Shape Too Early

Initially, I thought my main issue was:

```text
Expected:
word | nentry

Mine:
bull | bear
```

and I started thinking about pivoting or unpivoting.

However, I eventually realized:

The output structure was not the difficult part.

The difficult part was correctly counting exact word occurrences.

This was a useful reminder that:

> Before worrying about formatting the output, verify that the underlying calculation is correct.

---

## StrataScratch Solution

```sql
SELECT
    'bull' AS word,
    SUM(
        (
            LENGTH(LOWER(contents))
            -
            LENGTH(
                REPLACE(
                    LOWER(contents),
                    ' bull ',
                    ''
                )
            )
        )
        /
        LENGTH(' bull ')
    ) AS nentry
FROM google_file_store

UNION ALL

SELECT
    'bear' AS word,
    SUM(
        (
            LENGTH(LOWER(contents))
            -
            LENGTH(
                REPLACE(
                    LOWER(contents),
                    ' bear ',
                    ''
                )
            )
        )
        /
        LENGTH(' bear ')
    ) AS nentry
FROM google_file_store;
```

---

## Understanding the LENGTH - REPLACE Trick

Suppose we have:

```text
bull bull bear
```

The original string length is:

```text
14
```

After removing:

```text
 bull
```

the string becomes shorter.

The difference in length tells us how many characters were removed.

Dividing by the length of the target word allows us to estimate the number of occurrences.

This is a clever SQL trick for counting substring occurrences.

---

## A Concern With the Official Solution

Although the solution passes the test cases, I believe it has a potential weakness.

It searches for:

```sql
' bull '
```

and:

```sql
' bear '
```

with spaces on both sides.

This assumes the word is surrounded by spaces.

### Example

Beginning of sentence:

```text
bull market is rising
```

The word:

```text
bull
```

does not have a leading space.

---

End of sentence:

```text
the market contains a bear
```

The word:

```text
bear
```

does not have a trailing space.

In these situations:

```sql
REPLACE(contents, ' bull ', '')
```

may fail to count valid occurrences.

The solution appears to assume the dataset is formatted in a way that avoids these edge cases.

---

## How I Would Improve It

One approach would be:

```sql
CONCAT(' ', LOWER(contents), ' ')
```

before applying the replacement logic.

This artificially creates word boundaries at the beginning and end of the string.

Another approach would be using regular expressions with word boundaries.

Conceptually:

```regex
\b bull \b
```

and

```regex
\b bear \b
```

which naturally handle:

* Beginning of string
* End of string
* Punctuation
* Multiple spaces

Unfortunately, text-processing capabilities in SQL are often limited compared to dedicated programming languages.

---

## SQL vs Python

One of my biggest takeaways from this problem was:

> Not every problem is naturally suited for SQL.

This problem is fundamentally:

```text
Text Processing
```

rather than:

```text
Relational Data Analysis
```

In Python, a solution using regular expressions would be much more straightforward and robust.

Example conceptually:

```python
re.findall(r'\bbull\b', text)
```

This immediately handles:

* Exact words
* Word boundaries
* Multiple occurrences
* Edge cases

without requiring string-length arithmetic.

---

## Key SQL Concepts Practiced

* String Functions
* LOWER()
* LENGTH()
* REPLACE()
* UNION ALL
* Text Processing
* Exact vs Partial Matching

---

## Mental Models

### SQL Is Strongest At

* Aggregations
* Joins
* Filtering
* Ranking
* Time-Series Analysis

### SQL Is Less Natural For

* Text Parsing
* Tokenization
* Regular Expressions
* Natural Language Processing

When a problem feels unusually awkward in SQL, it may be because the problem itself is not naturally relational.

---

## Future Reminder

When solving SQL problems, ask:

> Am I working with structured relational data, or am I trying to force a text-processing problem into SQL?

Sometimes the best lesson from a problem is learning which tool is most appropriate for the task.
