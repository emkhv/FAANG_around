
# Solving The Hardest HackerRank SQL Problem: 15 Days of Learning SQL

You can find the problem [here](https://www.hackerrank.com/challenges/15-days-of-learning-sql/problem).

## Problem Description

Julia conducted a 15-day SQL learning contest starting on March 1, 2016, and ending on March 15, 2016. Each day, hackers submitted solutions. We need to write a query to determine:
1. The total number of unique hackers who made at least one submission each day(for each day of the competition).
2. The hacker who made the maximum number of submissions on each day. If there are multiple hackers with the same maximum number of submissions, select the hacker with the lowest `hacker_id`.

The results should be sorted by submission date.

### Given Data

- **Hackers Table:**

| hacker_id | name   |
|-----------|--------|
| 79722     | Michael|
| 20703     | Angela |
| 36396     | Frank  |

- **Submissions Table:**

| submission_id | submission_date | hacker_id | score |
|---------------|-----------------|-----------|-------|
| 1             | 2016-03-01      | 79722     | 95    |
| 2             | 2016-03-02      | 20703     | 92    |

### Problem Constraints

- The query should print the unique hackers who made submissions each day, and the hacker who made the maximum submissions per day.
- If more than one hacker made the maximum submissions, choose the one with the lowest `hacker_id`.
- Join the results by the date.

### Example

Given the following input data, the output should be:

| submission_date | unique_hackers | hacker_id | name    |
|-----------------|----------------|-----------|---------|
| 2016-03-01      | 4              | 20703     | Angela  |
| 2016-03-02      | 2              | 79722     | Michael |
| 2016-03-03      | 2              | 20703     | Angela  |
| 2016-03-04      | 2              | 20703     | Angela  |
| 2016-03-05      | 1              | 36396     | Frank   |
| 2016-03-06      | 1              | 20703     | Angela  |

## Approach

To solve this problem, we will:
1. Use a recursive Common Table Expression (CTE) to track the hackers who made submissions each day from March 1 to March 15, 2016, using the `hacker_id`s from the previous day. (Note: Use `WITH RECURSIVE` for PostgreSQL, as this solution was originally submitted in Oracle).
2. Count how many submissions each hacker made for each day of the competition using the `COUNT()` window function.
3. Determine the hacker who made the maximum number of submissions each day, using the `RANK()` window function to rank the hackers based on their submission counts.
4. Join the results with the `Hackers` table to get the name of the hacker with the maximum submissions.


## SQL Query

```sql
WITH rec(hacker_id, submission_date) AS 
(
  SELECT hacker_id, CAST(submission_date AS DATE) AS submission_date
  FROM Submissions
  WHERE submission_date = TO_DATE('2016-03-01', 'YYYY-MM-DD')
  UNION ALL
  SELECT e.hacker_id, CAST(e.submission_date AS DATE) AS submission_date
  FROM Submissions e
  INNER JOIN rec r ON e.hacker_id = r.hacker_id
  AND CAST(e.submission_date AS DATE) = r.submission_date + 1
)
SELECT rec.submission_date, 
                COUNT(DISTINCT rec.hacker_id) AS unique_hackers, 
                most_sub.hacker_id, 
                hackers.name 
FROM rec
INNER JOIN 
(
  SELECT submission_date, MIN(hacker_id) AS hacker_id 
  FROM 
  (
    SELECT submission_date, hacker_id, 
           RANK() OVER (PARTITION BY submission_date ORDER BY num_submissions DESC) AS rank
    FROM 
    (
      SELECT submission_date, hacker_id, 
             COUNT(hacker_id) OVER (PARTITION BY submission_date, hacker_id) AS num_submissions
      FROM Submissions
    ) inner_query
  ) ranked_query 
  WHERE rank = 1 
  GROUP BY submission_date
) most_sub 
ON rec.submission_date = CAST(most_sub.submission_date AS DATE)
INNER JOIN hackers ON most_sub.hacker_id = hackers.hacker_id
GROUP BY rec.submission_date, most_sub.hacker_id, hackers.name 
ORDER BY rec.submission_date;
```



## Explanation

The hardest part of this problem is counting the number of unique hackers that made a submission each day, and determine it for each day of the competition, so if they missed their streak, then they should be out of the count. Also it can be a bit confusing as the hacker who made the maximum number of submissions might not be in number of unique hackers that made a submission each day, so the two calculations should be separate. How do we achieve that?

First, we use a recursive CTE (this is the second problem in this list which uses a recursive CTE, the first one you can find [here](https://github.com/emkhv/FAANG_around/blob/main/google_1_hard.md))․ They say recursive queries are computationally heavy, and you need to be careful to give it a termination statement, but they make the code fairly readable and much simpler then if one tried to do them without one. Some of the solutions I checked used custom functions(MySQL), which is of course readable, but requires a lot of computational power as well and complicates the query, compared to the simple recursive solution we used here.
Here's how we achieve that:

1. **Recursive CTE**: This tracks submissions for each hacker starting from March 1, 2016. The query recursively joins each day with the previous day's submissions to ensure we account for all submissions per hacker. The condition `INNER JOIN rec r ON e.hacker_id = r.hacker_id` ensures we only include hackers who made submissions on consecutive days.

2. **Hacker with Maximum Submissions**:
   - The `COUNT(hacker_id) OVER (PARTITION BY submission_date, hacker_id)` counts the number of submissions each hacker made on each day.
   - `RANK() OVER (PARTITION BY submission_date ORDER BY num_submissions DESC)` ranks hackers based on their submissions, with the hacker having the most submissions getting the highest rank.
   - `MIN(hacker_id) WHERE rank = 1` selects the hacker with the most submissions for each day. If there's a tie, the one with the lowest `hacker_id` is chosen.

3. **Join with Hackers Table**: We join with the `Hackers` table to get the hacker's name.

4. **Final Output**: The results are joined by `submission_date` and ordered by `submission_date` to ensure the output is sorted correctly.

## Alternative Solutions

- Another great [solution](https://www.hackerrank.com/rest/contests/master/challenges/15-days-of-learning-sql/hackers/VladD/download_solution?primary=true) involved grouping submissions by submission date and hacker ID, then self-joining the rows where the submission date was greater than or equal to the submission date in the second table.

  This way we could take the hacker for each day and add all the previous dates when they made a submission, then we would add to 1 the number of days that passed after '2016-03-01', to find out what was the actual day of the contest, then count the unique dates in the second table, and if the number of the unique dates for each hacker was the same as the current, actual day of the contest, then we would conclude that he submitted every day and kept the streak. The solution also handled determining the hacker with the maximum submissions for each day.
```sql
with Sub1 as (
    select s1.submission_date, s1.hacker_id,
    count (distinct s1.submission_id) as date_submissions,
    1 + datediff(day, 'March 1, 2016', s1.submission_date) as contest_day,
    count (distinct s2.submission_date) as submission_days
    from Submissions s1 join Submissions s2
        on s1.hacker_id = s2.hacker_id and s1.submission_date >= s2.submission_date
    group by s1.submission_date, s1.hacker_id
),
Sub2 as (
    select submission_date, hacker_id, date_submissions
    from Sub1 where submission_days = contest_day
),
Sub3a as (
    select submission_date, count(hacker_id) as hackers
    from Sub2 group by submission_date
),
Sub3b as (
    select submission_date, max(date_submissions) as max_submissions
    from Sub1 group by submission_date
),
Sub4 as (
    select s1.submission_date, s3.hackers,
        (select top 1 s2.hacker_id from Sub1 s2
         where s1.submission_date = s2.submission_date and s1.max_submissions = s2.date_submissions
         order by s2.hacker_id) as hacker_id
    from Sub3b s1
    join Sub3a s3 on s1.submission_date = s3.submission_date
)
select s.*, h.name from Sub4 s join Hackers h on s.hacker_id = h.hacker_id order by submission_date
```
- As always, I asked ChatGPT to solve it as well, but the answer was incorrect, so it will not be mentioned here.

---

### Connect with Me:
- **LeetCode Profile:** [emkhv](https://leetcode.com/emkhv/)
- **LinkedIn:** [emkhv](https://www.linkedin.com/in/emkhv/)

Thank you for checking out the [FAANG Around series](https://github.com/emkhv/FAANG_around/). I hope these solutions help you in your SQL journey. Feel free to explore the provided SQL queries, try them out, and reach out for further discussions or clarifications.
