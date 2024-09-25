# 🍜 Case Study #1: Danny's Diner 
<img src="https://user-images.githubusercontent.com/81607668/127727503-9d9e7a25-93cb-4f95-8bd0-20b87cb4b459.png" alt="Image" width="450" height="520">

## 📚 Table of Contents
- 📈 [Business Task](#business-task)
- 🏷️ [Entity Relationship Diagram](#entity-relationship-diagram)
- 💡 [Question and Solution](#question-and-solution)


It is important to mention that all the data related to the case study has been referenced from the following link: [here](https://8weeksqlchallenge.com/case-study-1/).

## Business Task
Danny seeks to use the data to answer key questions about his restaurant's customers, focusing on how often they visit, how much they spend, and which dishes they enjoy most. 🍣🍛🍜

***

## Entity Relationship Diagram

![image](https://user-images.githubusercontent.com/81607668/127271130-dca9aedd-4ca9-4ed8-b6ec-1e1920dca4a8.png)

***

## Question and Solution

**1. What is the total amount each customer spent at the restaurant?**

````sql
SELECT 
  sales.customer_id, 
  SUM(menu.price) AS total_sales
FROM 
  dannys_diner.sales
INNER JOIN 
  dannys_diner.menu
ON 
  sales.product_id = menu.product_id
GROUP BY 
  sales.customer_id
ORDER BY 
  sales.customer_id ASC;
````
### Step 1: Join the Tables
Use INNER JOIN to combine the dannys_diner.sales and dannys_diner.menu tables on the product_id column, which is common to both tables.

### Step 2: Calculate Total Sales
Apply the SUM function to the menu.price column to calculate the total sales amount for each customer.

### Step 3: Group and Order Results
Group the results by sales.customer_id to aggregate the sales per customer, and then order the output by sales.customer_id in ascending order.

### Answer:
| customer_id | total_sales |
| ----------- | ----------- |
| A           | 76          |
| B           | 74          |
| C           | 36          |

This shows the customer_id alongside the total_sales for each customer, based on the prices of the products they purchased.

- Customer A spent $76.
- Customer B spent $74.
- Customer C spent $36.

***

**2. How many days has each customer visited the restaurant?**

````sql
SELECT 
  customer_id, 
  COUNT(DISTINCT order_date) AS visit_count
FROM dannys_diner.sales
GROUP BY customer_id;
````

#### Steps:
- SELECT Statement: We begin by selecting two columns: customer_id and the count of distinct order_date.
- COUNT(DISTINCT order_date): This function counts the unique dates on which each customer has placed an order, thereby representing their visit days.
- FROM Clause: The data is sourced from the dannys_diner.sales table, which contains records of customer orders.
- GROUP BY Clause: Finally, we group the results by customer_id to ensure that the visit count is calculated for each customer individually.

#### Answer:
| customer_id | visit_count |
| ----------- | ----------- |
| A           | 4          |
| B           | 6          |
| C           | 2          |

- Customer A visited 4 times.
- Customer B visited 6 times.
- Customer C visited 2 times.

***

**3. What was the first item from the menu purchased by each customer?**

````sql
WITH ordered_sales AS (
  SELECT 
    s.customer_id, 
    s.order_date, 
    m.product_name,
    DENSE_RANK() OVER (
      PARTITION BY s.customer_id 
      ORDER BY s.order_date
    ) AS rank
  FROM 
    dannys_diner.sales s
  INNER JOIN 
    dannys_diner.menu m ON s.product_id = m.product_id
)

SELECT DISTINCT
  customer_id, 
  product_name
FROM 
  ordered_sales
WHERE 
  rank = 1;
````

#### Step-by-Step Breakdown:
Step 1: Organize and Rank Purchases
First, The WITH clause to create a temporary table called ordered_sales to organize all the sales:
````sql
WITH ordered_sales AS (
  SELECT 
    s.customer_id, 
    s.order_date, 
    m.product_name,
    DENSE_RANK() OVER (
      PARTITION BY s.customer_id 
      ORDER BY s.order_date
    ) AS rank
  FROM 
    dannys_diner.sales s
  INNER JOIN 
    dannys_diner.menu m ON s.product_id = m.product_id
)
````
Here’s what’s happening in this part:

customer_id: I'm grabbing the customer ID from the sales table.
order_date: This helps me determine the order in which the customer made their purchases.
product_name: I join the menu table to get the names of the products the customer bought.
DENSE_RANK(): This function is key because it lets me rank each customer’s purchases by their visit date. The first visit gets rank 1, and any items bought during that visit share the same rank.

Step 2: Filtering for First Visits

Now that I’ve ranked each customer’s purchases by date, I’ll filter out only those purchases that occurred on their first visit (rank = 1). I use DISTINCT to make sure I don’t list the same item multiple times if they bought it more than once on that visit.

````sql
SELECT DISTINCT
  customer_id, 
  product_name
FROM 
  ordered_sales
WHERE 
  rank = 1;
````
DISTINCT: This ensures that if a customer bought the same item twice on their first visit, it only shows up once.
WHERE rank = 1: I’m filtering to only show the products that were purchased on the first visit.

This query returns a list of customers along with the unique items they bought on their first visit. If a customer bought multiple items on that visit, all of those will be shown. However, the same item won’t be repeated if purchased multiple times on the same day.

#### Answer:
| customer_id | product_name | 
| ----------- | ----------- |
| A           | curry        | 
| A           | sushi        | 
| B           | curry        | 
| C           | ramen        |

- Customer A placed an order for both curry and sushi simultaneously, making them the first items in the order.
- Customer B's first order is curry.
- Customer C's first order is ramen.

***

**4. What is the most purchased item on the menu and how many times was it purchased by all customers?**

````sql
SELECT 
  menu.product_name,
  COUNT(sales.product_id) AS most_purchased_item
FROM dannys_diner.sales
INNER JOIN dannys_diner.menu
  ON sales.product_id = menu.product_id
GROUP BY menu.product_name
ORDER BY most_purchased_item DESC
LIMIT 1;
````

#### Steps:

Using COUNT() to calculate how many times each product was purchased and then sort the results using ORDER BY in descending order. The LIMIT 1 clause ensures that only the most purchased item is retrieved. This allows us to quickly find the most popular menu item and how often it was ordered.

#### Answer:
| most_purchased | product_name | 
| ----------- | ----------- |
| 8       | ramen |


***

**5. Which item was the most popular for each customer?**

````sql
WITH most_popular AS (
  SELECT 
    sales.customer_id, 
    menu.product_name, 
    COUNT(menu.product_id) AS order_count,
    DENSE_RANK() OVER (
      PARTITION BY sales.customer_id 
      ORDER BY COUNT(sales.customer_id) DESC) AS rank
  FROM dannys_diner.menu
  INNER JOIN dannys_diner.sales
    ON menu.product_id = sales.product_id
  GROUP BY sales.customer_id, menu.product_name
)

SELECT 
  customer_id, 
  product_name, 
  order_count
FROM most_popular 
WHERE rank = 1;
````

#### Steps:
1. Common Table Expression (CTE) most_popular:
This part of the query calculates how many times each customer ordered each item using the COUNT() function on menu.product_id.
I use the DENSE_RANK() window function to rank the items in descending order of how often they were ordered (ORDER BY COUNT(sales.customer_id) DESC).
The PARTITION BY sales.customer_id ensures that the ranking is reset for each customer, so the items are ranked separately for every customer.
2. Main Query:
The main query then filters the most_popular CTE to only return the item with a rank = 1 for each customer, which corresponds to their most frequently purchased item.


#### Answer:
| customer_id | product_name | order_count |
| ----------- | ---------- |------------  |
| A           | ramen        |  3   |
| B           | sushi        |  2   |
| B           | curry        |  2   |
| B           | ramen        |  2   |
| C           | ramen        |  3   |

Customer A purchased ramen 3 times, making it their most popular item.
Customer B has three items (sushi, curry, and ramen) that were all purchased 2 times each. Since all of these have the same purchase count, they are tied as the most popular items for Customer B.
Customer C purchased ramen 3 times, which is their most frequently ordered item.

***

**6. Which item was purchased first by the customer after they became a member?**

```sql
WITH joined_as_member AS (
  SELECT
    members.customer_id, 
    sales.product_id,
    ROW_NUMBER() OVER (
      PARTITION BY members.customer_id
      ORDER BY sales.order_date) AS row_num
  FROM dannys_diner.members
  INNER JOIN dannys_diner.sales
    ON members.customer_id = sales.customer_id
    AND sales.order_date > members.join_date
)

SELECT 
  customer_id, 
  product_name 
FROM joined_as_member
INNER JOIN dannys_diner.menu
  ON joined_as_member.product_id = menu.product_id
WHERE row_num = 1
ORDER BY customer_id ASC;
```

WITH clause (joined_as_member):

This part of the query creates a temporary result set where we join the members and sales tables.
We use the INNER JOIN to link members with sales based on the customer_id.
The additional condition sales.order_date > members.join_date ensures we only consider sales that happened after the customer joined as a member.
We then use the ROW_NUMBER() window function to assign a unique row number (row_num) to each purchase for each customer, ordered by the order_date (oldest to newest). The PARTITION BY ensures that the row numbering is reset for each customer.
Main Query:

After creating the temporary table, we join it with the menu table to get the product_name corresponding to the product_id from the sales data.
The WHERE row_num = 1 clause ensures we only select the first purchase (the one with the lowest row number) for each customer.
Finally, the ORDER BY customer_id ASC sorts the results by customer ID.

#### Answer:
| customer_id | product_name |
| ----------- | ---------- |
| A           | ramen        |
| B           | sushi        |

- Customer A's first order as a member is ramen.
- Customer B's first order as a member is sushi.

***

**7. Which item was purchased just before the customer became a member?**

````sql
WITH purchases_before_membership AS (
  SELECT
    members.customer_id, 
    sales.product_id,
    ROW_NUMBER() OVER (
      PARTITION BY members.customer_id
      ORDER BY sales.order_date DESC) AS row_num
  FROM dannys_diner.sales
  INNER JOIN dannys_diner.members
    ON sales.customer_id = members.customer_id
  WHERE sales.order_date < members.join_date
)

SELECT 
  customer_id, 
  product_name
FROM purchases_before_membership
INNER JOIN dannys_diner.menu
  ON purchases_before_membership.product_id = menu.product_id
WHERE row_num = 1
ORDER BY customer_id ASC;
````

The query uses ROW_NUMBER() to rank purchases made before the customer became a member based on the order_date, ensuring that we only retrieve the last purchase.
The order_date is still used for ordering within the WITH clause, but it is not included in the final output.
The final result contains only the customer_id and the product_name of the item purchased just before becoming a member.

#### Output:

| customer_id | product_name |
| ----------- | ---------- |
| A           | sushi        |
| B           | sushi        |


Customer A and Customer B both purchased sushi as their last item before joining the membership.

***

**8. What is the total items and amount spent for each member before they became a member?**
