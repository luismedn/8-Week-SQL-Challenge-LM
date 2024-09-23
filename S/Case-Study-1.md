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
