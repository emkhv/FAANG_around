# FAANGiing Around - SQL Solutions for FAANG Interview Questions
---

Welcome to my journey of conquering SQL challenges, particularly those dreaded FAANG interview questions.

My name is Emma Khachatryan. I've been learning SQL for a year now. My journey started with data engineering, so DDL was a main focus for me for a while, but recently I decided to challenge myself and deepen my knowledge in DML `SELECT` statement.

At each step of the learning process, there comes a short period when you think you already learned the majority of the information about a topic. I myself often get trapped in that mindset. Then some encounters or discussions bring me back to the track of learning, as I realize there is more to every process in data analytics/science than I realize.

In SQL, you can have the same output coming from completely differently structured queries, and these series are focused on that.

I'll walk you through each problem, providing constraints, example output, and more. Then, I'll unveil my approach and solution. Additionally, I'll guide you to other solutions I've discovered and share some unique methods that caught my attention.

It's important to note that the solutions presented here prioritize education over efficiency. One of the principles I use when writing the queries is **avoiding unnecessary window functions**. In the future maybe I should discuss the window functions more in depth. I found using a `JOIN` can pretty much replicate any window function, even the `"running_total"`. You will find many **advanced joins** used. Don't be fooled though; there are advanced functions like `WITH RECURSIVE` and `PERCENTILE_CONT() WITHIN GROUP()`.


## Series Overview:


### 1. AWS Server Fleet Optimization

#### Problem Statement:
[Calculate the total time the AWS server fleet was running, given start and stop times of individual servers. Output the result in units of full days.]

#### Constraints:
- Each server might start and stop several times.
- The total time in which the server fleet is running can be calculated as the sum of each server's uptime.

#### Example Output:
|total_uptime_days|
|-----------------|
|21|

#### My Approach:
- Utilized advanced joins to handle multiple start-stop cycles of servers.
- Converted time intervals to EPOCH and calculated the sum in days.

#### Alternative Solutions:
- Explored a solution using the `LEAD()` window function provided by the original site.

#### Additional Insights:
- Demonstrated the use of `EXTRACT` and `SUM` for efficient time calculations.
- Avoided window functions in the main solution.

#### SQL Query:
```sql
-- Calculate the total uptime of the server fleet in full days
WITH cte AS (
    SELECT
        s1.status_time AS start_time,
        MIN(s2.status_time) AS stop_time
    FROM
        server_utilization s1
    INNER JOIN
        server_utilization s2 ON s1.server_id = s2.server_id
                               AND s2.session_status LIKE 'stop'
                               AND s1.status_time < s2.status_time
    WHERE
        s1.session_status LIKE 'start'
    GROUP BY
        s1.status_time
)
SELECT
    SUM(EXTRACT(EPOCH FROM (stop_time - start_time)) / 86400)::INT AS total_uptime_days
FROM
    cte;
```
##### Problem Link: [AWS Server Fleet Optimization](https://datalemur.com/questions/total-utilization-time)

---

### 2. Google Interview Question: Finding Median Searches

#### Problem Statement:
[Find the median number of searches a person made last year, given a summary table with the number of searches and the corresponding number of users.]

#### Constraints:
- Round the median to one decimal point.

#### Example Output:
|median|
|------|
|2.5|

#### My Approach:
- Used a recursive common table expression (CTE) to expand the search frequency table.
- Calculated the median using the `PERCENTILE_CONT` function and rounded the result.

#### Alternative Solutions:
- Explored a more concise solution using `GENERATE_SERIES` for table expansion.
- Found an alternative approach using `CROSS JOIN` and `ROW_NUMBER()` for median calculation.

#### Additional Insights:
- Experimented with various solutions, exploring different SQL techniques.
- Discussed the trade-offs and complexities of each solution.

#### SQL Query:
```sql
-- Calculate the median number of searches made by a user
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
##### Problem Link: [Google Interview Question: Finding Median Searches](https://datalemur.com/questions/median-search-freq)

## Connect with Me:
- **LeetCode Profile:** [emkhv](https://leetcode.com/emkhv/)
- **LinkedIn:** [emkhv](https://www.linkedin.com/in/emkhv/)

Thank you for joining me on my FAANGing Around SQL series. I hope these solutions and insights prove helpful in your SQL journey. Feel free to explore the provided SQL queries, try them out on your own, and don't hesitate to reach out for discussions, feedback, or further clarification.

Stay tuned for more FAANG interview questions and SQL solutions!


Happy coding!
