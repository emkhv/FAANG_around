# FAANG Around - SQL Solutions for FAANG Interview Questions
---

Welcome to my journey of conquering SQL challenges, particularly those dreaded FAANG interview questions.

My name is Emma Khachatryan. I've been learning SQL for a year now. My journey started with data engineering, so DDL was a main focus for me for a while, but recently I decided to challenge myself and deepen my knowledge of DML's `SELECT` statement.

At each step of the learning process, there comes a short period when you think you already learned the majority of the information about a topic. I often get trapped in that mindset. Then some encounters or discussions bring me back to the track of learning, as I realize there is more to every process in data analytics/science than I realize.

In SQL, you can have the same output coming from completely differently structured queries, and these series are focused on that.

I'll walk you through each problem, providing constraints, example output, and more. Then, I'll unveil my approach and solution. Additionally, I'll guide you to other solutions I've discovered and share some unique methods that caught my attention.

It's important to note that the solutions presented here prioritize education over efficiency. One of the principles I use when writing the queries is **avoiding unnecessary window functions**. In the future maybe I should discuss the window functions more in depth. I found using a `JOIN` can pretty much replicate any window function, even the `"running_total"`. You will find many **advanced joins** used. Don't be fooled though; there are advanced functions like `WITH RECURSIVE` and `PERCENTILE_CONT() WITHIN GROUP()`.


## Series Overview:


### [1. Google Interview Question: Finding Median Searches](https://github.com/emkhv/FAANGing_around/blob/main/google_1_hard.md)

#### Problem Statement:
[Find the median number of searches a person made last year, given a summary table with the number of searches and the corresponding number of users.]

#### Given Data `search_frequency` Table:

| searches | num_users |
|----------|-----------|
| 1        | 2         |
| 2        | 2         |
| 3        | 3         |
| 4        | 1         |

#### Constraints:
- Round the median to one decimal point.

#### Example Output:
|median|
|------|
|2.5|


### My Approach:
- Used a recursive common table expression (CTE) to expand the search frequency table.
- Calculated the median using the `PERCENTILE_CONT` function and rounded the result.

### Alternative Solutions:
- Explored a more concise solution using `GENERATE_SERIES` for table expansion.
- Found an alternative approach using `CROSS JOIN` and `ROW_NUMBER()` for median calculation.



#### Find out the [solution](https://github.com/emkhv/FAANGing_around/blob/main/google_1_hard.md)

##### Problem Link: [Google Interview Question: Finding Median Searches](https://datalemur.com/questions/median-search-freq)

---

### [2. AWS Server Fleet Optimization](https://github.com/emkhv/FAANGing_around/blob/main/Amazon_1_hard.md)

#### Problem Statement:
[Calculate the total time the AWS server fleet was running, given the start and stop times of individual servers. Output the result in units of full days.]

#### Given Data `server_utilization` Table:

| server_id | status_time           | session_status |
|-----------|-----------------------|-----------------|
| 1         | 08/02/2022 10:00:00  | start           |
| 1         | 08/04/2022 10:00:00  | stop            |
| 2         | 08/17/2022 10:00:00  | start           |
| 2         | 08/24/2022 10:00:00  | stop            |

#### Constraints:
- Each server might start and stop several times.
- The total time in which the server fleet is running can be calculated as the sum of each server's uptime.

#### Example Output:
|total_uptime_days|
|-----------------|
|21|

### My Approach:
- Utilized advanced joins to handle multiple start-stop cycles of servers.
- Converted time intervals to EPOCH and calculated the sum in days.

### Alternative Solutions:
- Explored a solution using the `LEAD()` window function provided by the original site.



#### Find out the [solution](https://github.com/emkhv/FAANGing_around/blob/main/Amazon_1_hard.md)


##### Problem Link: [AWS: Server Fleet Optimization](https://datalemur.com/questions/total-utilization-time)

---

### [3. Maximize Prime Item Inventory](https://github.com/emkhv/FAANGing_around/blob/main/Amazon_2_hard.md)

#### Problem Summary

[Amazon aims to maximize the number of items it can stock in a 500,000-square-foot warehouse. The goal is to prioritize prime items and use the remaining space for non-prime items. Your task is to write a query to determine the number of prime and non-prime items that can be stored in the warehouse. The output should include the item type (prime_eligible or not_prime) and the maximum number of items that can be stocked.]

#### Given Data `inventory` Table:

| Column Name    | Type    |
| -------------- | ------- |
| item_id        | integer |
| item_type      | string  |
| item_category  | string  |
| square_footage | decimal |

#### Example Input:

| item_id | item_type      | item_category       | square_footage |
|---------|-----------------|---------------------|-----------------|
| 1374    | prime_eligible  | mini refrigerator   | 68.00           |
| 4245    | not_prime       | standing lamp       | 26.40           |
| 2452    | prime_eligible  | television          | 85.00           |
| 3255    | not_prime       | side table          | 22.60           |
| 1672    | prime_eligible  | laptop              | 8.50            |

#### Example Output:

| item_type      | item_count |
| -------------- | ---------- |
| prime_eligible | 9285       |
| not_prime      | 6          |

#### Approach

The problem description can be complex and confusing. To simplify, we need to calculate the number of prime items that can be stored, determine the remaining space for non-prime items, and then calculate the number of non-prime items that can fit into that space. The approach involves using a Common Table Expression (CTE) named 'prime' to perform these calculations and then selecting and presenting the results.

### Alternative Solutions:
Provided by the site, slightly more readable. Utilized additional CTEs for clarity.

#### Find out the [solution](https://github.com/emkhv/FAANGing_around/blob/main/Amazon_2_hard.md)

##### Problem Link: [Google Interview Question: Finding Median Searches](https://datalemur.com/questions/prime-warehouse-storage)


---

## Connect with Me:
- **LeetCode Profile:** [emkhv](https://leetcode.com/emkhv/)
- **LinkedIn:** [emkhv](https://www.linkedin.com/in/emkhv/)

Thank you for joining me on my FAANG Around SQL series. I hope these solutions and insights prove helpful in your SQL journey. Feel free to explore the provided SQL queries, try them out on your own, and don't hesitate to reach out for discussions, feedback, or further clarification.

Stay tuned for more FAANG interview questions and SQL solutions!


Happy coding!
