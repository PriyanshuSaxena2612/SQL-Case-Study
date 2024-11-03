# Case Study 1: Danny's Diner

### 1. What is the total amount each customer spent at the restaurant?

````sql
SELECT sales.customer_id,
       SUM(menu.price) AS total_price
FROM sales
    LEFT JOIN menu
        ON sales.product_id = menu.product_id
GROUP BY sales.customer_id
ORDER BY sales.customer_id;
````

#### Answer:
| customer_id | total_price |
| ----------- | ----------- |
| A           | 76          |
| B           | 74          |
| C           | 36          |

***

### 2. How many days has each customer visited the restaurant?

````sql
SELECT sales.customer_id,
       COUNT(DISTINCT sales.order_date) AS number_of_days_visit
FROM sales
GROUP BY sales.customer_id;
````

#### Answer:

| customer_id | number_of_days_visit |
| ----------- | -------------------- |
| A           | 4                    |
| B           | 6                    |
| C           | 2                    |

***

### 3. What was the first item from the menu purchased by each customer?

````sql
WITH product_rank
AS (SELECT sales.customer_id,
           sales.product_id,
           menu.product_name,
           DENSE_RANK() OVER (PARTITION BY sales.customer_id
                              ORDER BY sales.order_date,
                                       sales.product_id
                             ) AS products_ranked
    FROM sales
        LEFT JOIN menu
            ON sales.product_id = menu.product_id
   )
SELECT customer_id,
       product_name
FROM product_rank
WHERE products_ranked = 1;
````

#### Answer:
| customer_id | product_name | 
| ----------- | ----------- |
| A           | curry        | 
| A           | sushi        | 
| B           | curry        | 
| C           | ramen        |

***

### 4. What is the most purchased item on the menu and how many times was it purchased by all customers?

````sql
SELECT menu.product_name,
       COUNT(sales.product_id) AS number_of_purchases
FROM sales
    LEFT JOIN menu
        ON sales.product_id = menu.product_id
GROUP BY menu.product_name
ORDER BY count(sales.product_id) DESC
LIMIT 1;
````



#### Answer:
| product_name | number_of_purchases |
| ------------ | ------------------- |
| ramen        | 8                   |



***

### 5. Which item was the most popular for each customer?

````sql
WITH popular_rank
AS (SELECT sales.customer_id,
           sales.product_id,
           menu.product_name,
           COUNT(sales.product_id) as product_count,
           DENSE_RANK() OVER (PARTITION BY sales.customer_id
                              ORDER BY COUNT(sales.product_id) DESC
                             ) AS product_rank
    FROM sales
        LEFT JOIN menu
            ON sales.product_id = menu.product_id
    GROUP BY sales.customer_id,
             sales.product_id,
             menu.product_name
   )
SELECT customer_id,
       product_id,
       product_name
FROM popular_rank
WHERE product_rank = 1;
````

#### Answer:
| customer_id | product_id | product_name |
| ----------- | ---------- | ------------ |
| A           | 3          | ramen        |
| B           | 3          | ramen        |
| B           | 1          | sushi        |
| B           | 2          | curry        |
| C           | 3          | ramen        |


***

### 6. Which item was purchased first by the customer after they became a member?

````sql
WITH cte AS(
  SELECT 
    sales.customer_id, 
    members.join_date, 
    sales.order_date AS first_order_date, 
    sales.product_id, 
    menu.product_name, 
    DENSE_RANK() OVER(
      PARTITION BY sales.customer_id 
      ORDER BY 
        sales.order_date
    ) AS date_rank 
  FROM 
    sales 
    LEFT JOIN menu ON sales.product_id = menu.product_id 
    LEFT JOIN members ON sales.customer_id = members.customer_id 
  WHERE 
    members.join_date <= sales.order_date 
  ORDER BY 
    sales.customer_id
) 
SELECT 
  customer_id, 
  join_date, 
  first_order_date, 
  product_name 
from 
  cte 
WHERE 
  date_rank = 1;

````


#### Answer:
| customer_id | join_date  | first_order_date | product_name |
| ----------- | ---------- | ---------------- | ------------ |
| A           | 2021-01-07 | 2021-01-07       | curry        |
| B           | 2021-01-09 | 2021-01-11       | sushi        |

***

### 7. Which item was purchased just before the customer became a member?

````sql
WITH cte AS(
  SELECT 
    sales.customer_id, 
    members.join_date, 
    sales.order_date, 
    sales.product_id, 
    menu.product_name, 
    DENSE_RANK() OVER(
      PARTITION BY sales.customer_id 
      ORDER BY 
        sales.order_date DESC
    ) AS date_rank 
  FROM 
    sales 
    LEFT JOIN menu ON sales.product_id = menu.product_id 
    LEFT JOIN members ON sales.customer_id = members.customer_id 
  WHERE 
    members.join_date > sales.order_date 
  ORDER BY 
    sales.customer_id
) 
SELECT 
  customer_id, 
  join_date, 
  order_date, 
  product_name 
FROM 
  cte 
WHERE 
  date_rank = 1;
````

#### Answer:
| customer_id | join_date  | order_date | product_name |
| ----------- | ---------- | ---------- | ------------ |
| A           | 2021-01-07 | 2021-01-01 | sushi        |
| A           | 2021-01-07 | 2021-01-01 | curry        |
| B           | 2021-01-09 | 2021-01-04 | sushi        |

***

### 8. What is the total items and amount spent for each member before they became a member?

````sql
SELECT 
  sales.customer_id, 
  COUNT(sales.product_id) AS product_count, 
  SUM(menu.price) AS total_amount 
FROM 
  sales 
  LEFT JOIN menu ON sales.product_id = menu.product_id 
  LEFT JOIN members ON sales.customer_id = members.customer_id 
WHERE 
  sales.order_date < members.join_date 
GROUP BY 
  sales.customer_id;
````


#### Answer:
| customer_id | product_count | total_amount |
| ----------- | ------------- | ------------ |
| B           | 3             | 40           |
| A           | 2             | 25           |


***

### 9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier — how many points would each customer have?

````sql
SELECT 
  sales.customer_id, 
  SUM(
    CASE WHEN sales.product_id = 1 THEN menu.price * 20 ELSE menu.price * 10 END
  ) AS total_points 
FROM 
  sales 
  LEFT JOIN menu ON sales.product_id = menu.product_id 
GROUP BY 
  sales.customer_id 
ORDER BY 
  sales.customer_id;
````


#### Answer:
| customer_id | total_points |
| ----------- | ------------ |
| A           | 860          |
| B           | 940          |
| C           | 360          |


***

### 10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi — how many points do customer A and B have at the end of January?

````sql
SELECT 
  sales.customer_id, 
  SUM(
    CASE WHEN (
      sales.order_date - members.join_date BETWEEN 0 
      and 7
    ) 
    OR (sales.product_id = 1) THEN menu.price * 20 ELSE menu.price * 10 END
  ) AS total_points 
FROM 
  sales 
  LEFT JOIN menu ON sales.product_id = menu.product_id 
  LEFT JOIN members ON sales.customer_id = members.customer_id 
WHERE 
  sales.order_date >= members.join_date 
  AND sales.order_date <= CAST ('2021-01-31' AS DATE) 
GROUP BY 
  sales.customer_id 
ORDER BY 
  sales.customer_id;
````

#### Answer:
| customer_id | total_points |
| ----------- | ------------ |
| A           | 1020         |
| B           | 440          |


***

