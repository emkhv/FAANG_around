# Solving Google Interview Question: Finding Median Searches
You can find the problem following the [link](https://datalemur.com/questions/median-search-freq).

## Problem Description

Google's marketing team is making a Superbowl commercial and needs a simple statistic to put on their TV ad: the median number of searches a person made last year.

However, at Google scale, querying the 2 trillion searches is too costly. Luckily, you have access to the summary table which tells you the number of searches made last year and how many Google users fall into that bucket.

Write a query to report the median of searches made by a user. Round the median to one decimal point.

## Given Data

### `search_frequency` Table:

| searches | num_users |
|----------|-----------|
| 1        | 2         |
| 2        | 2         |
| 3        | 3         |
| 4        | 1         |

## Problem Constraints

- Find the median of searches made by a user.
- Round the median to one decimal point.
  
### Example

By expanding the `search_frequency` table, we get the list `[1, 1, 2, 2, 3, 3, 3, 4]`, which has a median of `2.5` searches per user.

| searches |
|----------|
| 2.5 |

## Approach

To find the median, we need to 
  - expand the `search_frequency` table
  - calculate the median from the resulting list. Very simple!

## SQL Query

Here's the SQL query to solve the problem:

```sql
WITH RECURSIVE cte_numbers(searches, num_users) 
AS (
    SELECT 
        searches, num_users
    FROM 
        search_frequency
    UNION ALL
    SELECT    
        searches, num_users - 1
    FROM    
        cte_numbers
    WHERE 
        num_users > 1
)
SELECT 
    ROUND(PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY searches)::NUMERIC, 1) AS "Percentile_Cont"
FROM 
    cte_numbers;
```

## Explanation

The query uses a recursive common table expression (CTE) to generate the expanded list of searches. 
The `PERCENTILE_CONT` function is then used to calculate the median, and the result is rounded to one decimal point.




## P.S.

This was my first attempt in constructing a RECURSIVE query. The way I imagine it is that I can write a `UNION ALL` but give it a condition that will make it dinamic.
After solving it I looked through for other solutions. 

This is the solution __the site__ provides, which in it's turn is very slick 
```sql
WITH searches_expanded AS (
  SELECT searches
  FROM search_frequency
  GROUP BY 
    searches, 
    GENERATE_SERIES(1, num_users))

SELECT 
  ROUND(PERCENTILE_CONT(0.50) WITHIN GROUP (
    ORDER BY searches)::DECIMAL, 1) AS  median
FROM searches_expanded;
```
As you can see the only difference is the use of `GENERATE_SERIES(1, num_users)` which is a rarely used technique, and is usually used in DDL, when generating an index for a table.


Another version was the one I got from __AI__. This is not the only output it gave, but the other one was even longer, so I didn't include it.

```sql
WITH expanded_searches AS (
    SELECT searches
    FROM search_frequency
    CROSS JOIN generate_series(1, num_users) AS users
    ORDER BY searches
),

sorted_searches AS (
    SELECT searches,
           ROW_NUMBER() OVER (ORDER BY searches) AS row_num,
           COUNT(*) OVER () AS total_rows
    FROM expanded_searches
)

SELECT ROUND(AVG(searches)::numeric, 1) AS median
FROM sorted_searches
WHERE row_num BETWEEN (total_rows + 1) / 2 AND (total_rows + 2) / 2;
```
The majority of the solutions I saw were very overcomplicated like this. Although this is an interesting approach to consider.

Thank you for joining me in my first FAANGing around series. Have nice day!
 


