# AWS Server Fleet Optimization
You can find the problem following the [link](https://datalemur.com/questions/total-utilization-time).
## Problem Description

Amazon Web Services (AWS) is powered by fleets of servers, and senior management has requested data-driven solutions to optimize server usage. 
The task is to write a query that calculates the total time that the fleet of servers was running. The output should be in units of full days.

## Assumptions

- Each server might start and stop several times.
- The total time in which the server fleet is running can be calculated as the sum of each server's uptime.

## Given Data

### `server_utilization` Table:

| server_id | status_time           | session_status |
|-----------|-----------------------|-----------------|
| 1         | 08/02/2022 10:00:00  | start           |
| 1         | 08/04/2022 10:00:00  | stop            |
| 2         | 08/17/2022 10:00:00  | start           |
| 2         | 08/24/2022 10:00:00  | stop            |

## Problem Constraints

- Calculate the total time the server fleet was running.
- Output the result in units of full days.

## SQL Query

Here's my solution the the problem:

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
## Explanation

We had 2 constraints to overcome:

__Each server might start and stop several times__

To calculate the sum of all the hours the fleet of servers was running, we need to consieder that simple `max()` and `min()` functions will not work. 
We need the data in a specific form to be able to aggregate the right output.  
I had the idea of displaying the data horizontally by grouping them by the server_id, as the result I will have 1 row, displaying both the start and the e. 
I used _advanced joins_ like `on s2.session_status LIKE 'stop'` and `on s1.status_time < s2.status_time`.

__Output the result in units of full days__

This part was fairly easy, to sum the difference and display in days: 

On the one hand you want to extract the days from the sum of intervals you have, but there is a problem, that the hours and days are summed separately:
You have to convert everything to EPOCH which is basically a countdown, then divide it to 84000 as if seconds in days.

We did it using `SUM(EXTRACT(EPOCH FROM (stop_time - start_time)) / 86400)::INT`.

## Result

The result of the query should be the total uptime of the server fleet, expressed in full days.

### Example Output:

|total_uptime_days|
|-----------------|
|21|

## Conclusion
The solution leverages SQL functions, such as `MIN`, `MAX`, and `EXTRACT`, to efficiently calculate the total uptime of the server fleet. 
The query considers the start and stop times for each server, providing a data-driven approach to optimize server usage.

## P.S. 

There were other soutions I visited after solving. The first one was the one the actual site provided.
```sql
WITH running_time 
AS (
  SELECT
    server_id,
    session_status,
    status_time AS start_time,
    LEAD(status_time) OVER (
      PARTITION BY server_id
      ORDER BY status_time) AS stop_time
  FROM server_utilization
)

SELECT
  DATE_PART('days', JUSTIFY_HOURS(SUM(stop_time - start_time))) AS total_uptime_days
FROM running_time
WHERE session_status = 'start'
  AND stop_time IS NOT NULL;
```

The `lead() over()` function is more readable. I wanted to avoid using any window function when solving it, so it was not part of mine.

