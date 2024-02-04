# Maximize Prime Item Inventory

Amazon wants to maximize the number of items it can stock in a 500,000 square feet warehouse. It wants to stock as many prime items as possible, and afterward use the remaining square footage to stock the most number of non-prime items.

## Assumptions

- Prime and non-prime items have to be stored in equal amounts, regardless of their size or square footage. This implies that prime items will be stored separately from non-prime items in their respective containers, but within each container, all items must be in the same amount.
- Non-prime items must always be available in stock to meet customer demand, so the non-prime item count should never be zero.
- Item count should be whole numbers (integers).

## Given Data

### `inventory` Table:

| Column Name    | Type    |
| -------------- | ------- |
| item_id        | integer |
| item_type      | string  |
| item_category  | string  |
| square_footage | decimal |
### __Example input__:

| item_id | item_type      | item_category       | square_footage |
|---------|-----------------|---------------------|-----------------|
| 1374    | prime_eligible  | mini refrigerator   | 68.00           |
| 4245    | not_prime       | standing lamp       | 26.40           |
| 2452    | prime_eligible  | television          | 85.00           |
| 3255    | not_prime       | side table          | 22.60           |
| 1672    | prime_eligible  | laptop              | 8.50            |

**Example Output:**

| item_type      | item_count |
| -------------- | ---------- |
| prime_eligible | 9285       |
| not_prime      | 6          |

## Approach

Usually when I write the 'Approach' section, I try to be as simple as possible, as it is very easy to understand the rest after you know the simple structure of the query intended.
In this case, the question itself is not simple or 'direct' enough for me to assume you know what it is actually 'saying'. 


I was struggling to understand the reqirements, and tried various solutions. 
I checked the comments, turned out I was not alone and many people were struggling with the description. I was planning to solve it in a day max, ended up solving in 2. 
And the final hint I got was from my friend who is a senior engineer and understood the problem much faster then I did(I am a junior I guess?) 


This was a very annoying problem, and I think they did it on purpose. I did the "AI-check", it couldn't solve it. 
If I happened to see the solution before the problem, I would imagine a very different problem description.

Anyway, let's tackle the solution approach.

The first thing you need to understand is that all the items of the Prime and Non-Prime lists CAN fit into the warehouse. 
So you can keep them all together and in multiple counts. We need to prioritise the Prime items. 

### Steps:


**`prime` cte:**
 - Count the sum of the square feet all Prime items are taking together
 - See how many times we can fit that sum in the warehouse, hence finding the `prime_item_count`(Now we know how many of each prime item we need to take).
 - Count how much space is left for the Non-Prime items.(Yes, all non-prime items CAN fit in this space, this is bizzare to me as well)


**Main query:**
 
 (We are going to use the `prime` cte in this one, but I will not join it in the `FROM` statement as expected.
Insted, we are going to extract everything separately, using it as a warehouse lol.)
We expect two rows to pop as an output and we already have the data they will display, so we can just select the string we need directrly. 
This is a very widely used technique.As I understand when there is a big calculation invoved and you don't want to overcomplicate the query, this method makes it both readable and light.

 - Select 'prime_eligible' string
 - Count all prime items that will be stored
 - Union 'not_prime' row
 - Select 'not_prime' string
 - Count all the not_prime items that will be stored(Here we use the second part of the cte `prime`, as we store the `space_left` there, we will
1. Count the sum of the square feet all Non-Prime items are taking together
2. See how many times we can fit that sum in the warehouse, hence finding the `non_prime_item_count`
3. Multiply it by the total_count
   )

That's all I guess, looks much simple. That desscription was a disaster :v 



## SQL Query

Here's the SQL query to solve the problem:

```sql
-- Create a Common Table Expression (CTE) named 'prime' to calculate the number of prime items and the remaining space
WITH prime AS (
    SELECT
        FLOOR(500000 / SUM(square_footage)) AS prime_item_count, 
        500000 - (SUM(square_footage) * FLOOR(500000 / SUM(square_footage))) AS space_left
    FROM
        inventory
    WHERE
        item_type LIKE 'prime_eligible'
)

-- Select prime eligible items and calculate the overall count of prime items to be stored
SELECT
    'prime_eligible' AS item_type,
    (SELECT prime_item_count FROM prime) * COUNT(DISTINCT item_id) AS total_items
FROM
    inventory
WHERE
    item_type LIKE 'prime_eligible'

UNION ALL -- Combine results with non-prime eligible items

-- Select non-prime eligible items and calculate the overall count of non-prime items to be stored
SELECT
    'not_prime' AS item_type,
    FLOOR((SELECT space_left FROM prime) / SUM(square_footage)) * COUNT(DISTINCT item_id) AS total_items
FROM
    inventory
WHERE
    item_type LIKE 'not_prime';
```

## Explanation

The query uses a Common Table Expression (CTE) named `prime` to calculate the number of prime items and the remaining space in the warehouse. It then selects prime and non-prime eligible items separately and calculates the overall count of items to be stored.

## Conclusion

The solution optimizes the storage of items in the warehouse, maximizing the number of prime items and utilizing the remaining space for non-prime items, meeting the specified assumptions and constraints.

---

### Connect with Me:
- **LeetCode Profile:** [emkhv](https://leetcode.com/emkhv/)
- **LinkedIn:** [emkhv](https://www.linkedin.com/in/emkhv/)

Thank you for joining me on [FAANG around series](https://github.com/emkhv/FAANG_around/). I hope these solutions and insights prove helpful in your SQL journey. Feel free to explore the provided SQL queries, try them out on your own, and don't hesitate to reach out for discussions, feedback, or further clarification.
