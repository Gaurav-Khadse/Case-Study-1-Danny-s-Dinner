# Case Study #1 - Danny's Diner👨🏻‍🍳

![image](https://github.com/user-attachments/assets/80afb41c-63e5-4035-9d84-3838c2946190)


## Contents
- [**Introduction**](#introduction)
- [**Problem Statement**](#problem-statement)
- [**Entity Relationship Diagram**](#entity-relationship-diagram)
- [**Case Study Questions & Solutions**](#case-study-questions--solutions)
- [**Bonus Questions & Solutions**](#bonus-questions--solutions)
- [**Key Insights**](#key-insights)


## Introduction

In early 2021, Danny follows his passion for Japanese food and opens "Danny's Diner," a charming restaurant offering sushi, curry, and ramen. However, lacking data analysis expertise, the restaurant struggles to leverage the basic data collected during its initial months to make informed business decisions. Danny's Diner seeks assistance in using this data effectively to keep the restaurant thriving.

## Problem Statement
Danny aims to utilize customer data to gain valuable insights into their visiting patterns, spending habits, and favorite menu items. By establishing a deeper connection with his customers, he can provide a more personalized experience for his loyal patrons.

He plans to use these insights to make informed decisions about expanding the existing customer loyalty program. Additionally, Danny seeks assistance in generating basic datasets for his team to inspect the data conveniently, without requiring SQL expertise.

Due to privacy concerns, he has shared a sample of his overall customer data, hoping it will be sufficient for you to create fully functional SQL queries to address his questions.

The case study revolves around three key datasets:

- Sales
- Menu
- Members
  
## Entity Relationship Diagram

![ER Diagram Dannys Diner](https://github.com/user-attachments/assets/4555228a-fc4b-418d-9200-a23710e2efe2)


## Case Study Questions & Solutions

1.What is the total amount each customer spent at the restaurant?

```sql
SELECT s.customer_id,SUM(m.price) AS total_amount
FROM sales s
JOIN menu m ON s.product_id = m.product_id
GROUP BY s.customer_id
ORDER BY s.customer_id;
```
- Answer:
  
  - The SQL query retrieves the customer_id and calculates the total amount spent (total_amnt) by each customer at the restaurant.
  - It combines data from the sales and menu tables based on matching product_id.
  - The results are grouped by customer_id.
  - The query then calculates the total sum of price for each group of sales records with the same customer_id.
  - Finally, the results are sorted in ascending order based on the customer_id.
  - Sub-point (indented with 2 spaces)
    

2.How many days has each customer visited the restaurant?

```sql

SELECT customer_id,COUNT(DISTINCT order_date) AS days_visited
FROM sales
GROUP BY customer_id;
```
- Answer:
  
  - The SQL query selects the customer_id and counts the number of distinct order dates (No_Days) for each customer.
  - It retrieves data from the sales table.
  - The results are grouped by customer_id.
  - The COUNT(DISTINCT order_date) function calculates the number of unique order dates for each customer.
  - Finally, the query presents the total number of unique order dates as No_Days for each customer.


3.What was the first item from the menu purchased by each customer?
```sql
WITH first_purchases AS (
SELECT s.customer_id,m.product_name,s.order_date,
DENSE_RANK() OVER (PARTITION BY s.customer_id ORDER BY s.order_date) AS rn
FROM sales s
JOIN menu m ON s.product_id = m.product_id
)
SELECT customer_id,product_name
FROM first_purchases
WHERE rn = 1;
```
- Answer:
  - The SQL query uses a Common Table Expression (CTE) named CTE to generate a temporary result set.
  - Within the CTE, it selects the customer_id, assigns a dense rank to each row based on the order date for each customer, and retrieves the corresponding product_name from the menu table.
  - The sales table is joined with the menu table on matching product_id.
  - The DENSE_RANK() function assigns a rank to each row within the partition of each customer_id based on the order_date in ascending order.
  - Each customer_id has its own partition and separate ranks based on the order dates of their purchases.
  - Next, the main query selects the customer_id and corresponding product_name from the CTE.
  - It filters the results and only includes rows where the rank rn is equal to 1, which means the earliest purchase for each customer_id.
  - As a result, the query returns the first purchased product for each customer.
 

4.What is the most purchased item on the menu and how many times was it purchased by all customers?
```sql
SELECT m.product_name,COUNT(*) AS purchase_count
FROM sales s
JOIN menu m ON s.product_id = m.product_id
GROUP BY m.product_name
ORDER BY purchase_count DESC
LIMIT 1;
```
- Answer:
  -  The SQL query selects the product_name from the menu table and counts the number of times each product was ordered (most_ordered).
  - It retrieves data from the Sales table and joins it with the menu table based on matching product_id.
  - The results are grouped by product_name.
  - The COUNT(*) function calculates the number of occurrences of each product_id in the Sales table.
  - The query then presents the product_name and its corresponding count as most_ordered for each product.
  - Next, the results are sorted in descending order based on the most_ordered column, so the most ordered product appears first.
  - The LIMIT 1 clause is used to restrict the result to only one row, effectively showing the most ordered product.


5.Which item was the most popular for each customer?
```sql
WITH customer_favorites AS (
SELECT s.customer_id,m.product_name,COUNT(*) AS purchase_count,
dense_rank() OVER (PARTITION BY s.customer_id ORDER BY COUNT(*) DESC) AS rn
FROM sales s
JOIN menu m ON s.product_id = m.product_id
GROUP BY s.customer_id, m.product_name
)
SELECT customer_id,product_name,purchase_count
FROM customer_favorites
WHERE rn = 1;
```
- Answer:
  - The SQL query uses a Common Table Expression (CTE) named CTE to generate a temporary result set.
  - Within the CTE, it selects the customer_id, product_name, and counts the number of times each product was ordered (purchase_count) for each customer.
  - It retrieves data from the sales table and joins it with the menu table based on matching product_id.
  - The results are grouped by customer_id and product_name to get the count of orders for each product of each customer.
  - The COUNT(*) function calculates the number of occurrences of each product_id in the menu table.
  - The DENSE_RANK() function assigns a rank to each row within the partition of each customer_id based on the order count of products in descending order.
  - Each customer_id has its own partition and separate ranks based on the number of product orders.
  - Next, the main query selects the customer_id, product_name, and purchase_count from the CTE.
  - It filters the results and only includes rows where the rank rn is equal to 1, which means the most ordered product for each customer_id.
  - As a result, the query returns the customer's ID, the most ordered product, and the number of times it was ordered by that customer.



6.Which item was purchased first by the customer after they became a member?
```sql
WITH first_post_member_purchase AS (
SELECT s.customer_id,m.product_name,
DENSE_RANK() OVER (PARTITION BY s.customer_id ORDER BY s.order_date) AS rn
FROM sales s
JOIN menu m ON s.product_id = m.product_id
JOIN members mem ON s.customer_id = mem.customer_id
WHERE s.order_date >= mem.join_date
)
SELECT * FROM first_post_member_purchase WHERE rn = 1;
```
- Answer:
  - The SQL query is designed to retrieve the first product purchased by each customer after becoming a member.
  - The query uses Common Table Expression (CTE) named first_post_member_purchase to organize intermediate results.
  - In the CTE, the sales table is aliased as s, the menu table as m, and the members table as mem.
  - The query performs INNER JOIN operations between:
  - sales and menu tables using the product_id,
  - sales and members tables using the customer_id.
  - The WHERE clause filters records to only include sales on or after the customer’s join date.
  - The DENSE_RANK() window function assigns a rank to each sale per customer, ordered by the sale date.
  - This helps to find the first qualifying purchase after a customer became a member.
  - The outer query selects only the rows where the rank rn = 1, meaning the first post-membership purchase.
  - The final result displays the customer_id, the product_name of their first purchase, and the rank rn.
  - Thus, the query returns a list of customers and the first product they ordered after joining the membership.


7.Which item was purchased just before the customer became a member?
```sql
WITH ranked_purchases AS (
SELECT s.customer_id,m.product_name,s.order_date,me.join_date,
ROW_NUMBER() OVER (PARTITION BY s.customer_id ORDER BY s.order_date DESC) AS rn
FROM sales AS s
INNER JOIN menu AS m ON s.product_id = m.product_id
INNER JOIN members AS me ON s.customer_id = me.customer_id
WHERE s.order_date < me.join_date
)
SELECT customer_id,product_name,order_date,join_date,rn
FROM ranked_purchases
WHERE rn = 1;

```
- Answer:
  - To find the item that was purchased just before a customer became a member, we use a common table expression (CTE) called ranked_purchases. 
  - This CTE joins the sales, menu, and members tables to gather information about each purchase, including the product name, purchase date, and membership join date. 
  - We only consider purchases that occurred before the customer’s join date using the condition s.order_date < me.join_date.
  - Inside the CTE, we use the ROW_NUMBER() window function to assign a rank to each qualifying purchase, ordered by the purchase date in descending order (ORDER BY s.order_date DESC).
  - This ensures that the most recent purchase before the customer became a member is ranked as 1.
  - Finally, in the main query, we select only those rows where rn = 1, meaning we retrieve the last purchase made before membership, along with customer ID, product name, order date, and join date.
  - This approach is reliable, accurate, and works well across different SQL databases.

8.What is the total items and amount spent for each member before they became a member?
```sql
SELECT s.customer_id,
COUNT(DISTINCT s.product_id) AS total_unique_items,
COUNT(*) AS total_purchases,
SUM(m.price) AS total_amount_spent
FROM sales s
JOIN menu m ON s.product_id = m.product_id
JOIN members mem ON s.customer_id = mem.customer_id
WHERE s.order_date < mem.join_date
GROUP BY s.customer_id
ORDER BY s.customer_id;

```
- Answer:
  - This SQL query calculates the total number of unique items, total purchases, and the total amount spent by each customer before they became a member.
  - The query uses INNER JOINs between the sales, menu, and members tables to get the necessary details for each purchase.
  - It filters the data using the condition s.order_date < mem.join_date, which ensures that only the purchases made before the membership start date are included.
  - The query groups the result by customer_id so that all metrics are calculated per customer.
  - COUNT(DISTINCT s.product_id) gives the number of unique items bought.
  - COUNT(*) gives the total number of transactions or purchases.
  - SUM(m.price) calculates the total amount spent.
  - Finally, the results are ordered by customer_id to keep the output clean and organized.


9.If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
```sql
SELECT s.customer_id,
SUM(CASE WHEN m.product_name = 'sushi' THEN m.price * 20  
ELSE m.price * 10                                 
END) AS total_points
FROM sales s
JOIN menu m ON s.product_id = m.product_id
GROUP BY s.customer_id
ORDER BY s.customer_id;
```
- Answer:
  - This query calculates how many reward points each customer earned based on their total spending.
  - Each $1 spent equals 10 points, but if the product purchased is sushi, then it earns double points (20 points per dollar).
  - The query joins the sales and menu tables using the product_id to access the price and product name for each sale.
  - It uses a CASE statement inside the SUM() function to apply the correct multiplier:
  - If the item is sushi, multiply price * 20.
  - Otherwise, multiply price * 10.
  - The result is grouped by customer_id to compute total points per customer.
  - The final result is sorted by customer_id.


10.In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?
```sql
SELECT s.customer_id,
SUM(CASE WHEN s.order_date BETWEEN mem.join_date AND DATE_ADD(mem.join_date, INTERVAL 6 DAY) THEN m.price * 20
WHEN m.product_name = 'sushi' THEN m.price * 20 ELSE m.price * 10 END) AS total_points
FROM sales s
JOIN menu m ON s.product_id = m.product_id
JOIN members mem ON s.customer_id = mem.customer_id
WHERE s.order_date <= '2021-01-31'  
GROUP BY s.customer_id
ORDER BY s.customer_id;
```
- Answer:
  - This query calculates the total points earned by each customer up to January 31, with the condition that the first 7 days after joining the program have a 2x points multiplier on all items, not just sushi.
  - The query performs a join between the sales, menu, and members tables.
  - The WHERE clause limits the results to orders placed on or before '2021-01-31'.
  - The CASE logic inside the SUM() function checks:
  - If the order date is between the join_date and 6 days after (inclusive), all items get price * 20.
  - If not in the first week but the product is sushi, still apply price * 20.
  - Otherwise, apply the standard price * 10.
  - The logic ensures both promotions are respected — one based on product type (sushi) and one based on the first week bonus.
  - The query groups by customer_id and returns total points earned up to the end of January.
  - It shows how promotions affect loyalty points for each member.

## Bonus Questions & Solutions

Join All The Things

```sql
WITH customer_member_status AS (
SELECT s.customer_id,s.order_date,m.product_name,m.price,
CASE WHEN mbr.join_date <= s.order_date THEN 'Y'ELSE 'N'END AS member
FROM sales s
INNER JOIN menu m ON s.product_id = m.product_id
LEFT JOIN members mbr ON s.customer_id = mbr.customer_id
)
SELECT customer_id,order_date,product_name,price,member
FROM customer_member_status
ORDER BY customer_id,member DESC,order_date;
```
- Answer:
  - The SQL query starts by creating a Common Table Expression (CTE) named customer_member_status.
  - Within the CTE, it selects customer_id, order_date, product_name, price, and uses a CASE statement to determine whether the customer is a member ('Y') or not ('N') based on their join date in the members table.
  - The sales table is aliased as s, the menu table as m, and the members table as mbr.
  - The query performs inner joins between sales and menu tables on matching product_id and left join between sales and members tables on matching customer_id.
  - For each row, the CASE statement checks if the join_date from the members table is less than or equal to the order_date from the sales table.
  - If true, it assigns 'Y' (member) to the member column, otherwise 'N' (non-member).
  - Next, the main query selects the customer_id, order_date, product_name, price, and member from the customer_member_status CTE.
  - Results are ordered first by customer_id in ascending order, then by member in descending order (so members appear first), and finally by order_date in ascending order.
  - The query presents the final results with the selected columns for each customer, showing whether they are a member or not for each order and sorted accordingly.


Rank All The Things


```sql
WITH customers_data AS (
SELECT sales.customer_id, sales.order_date,  menu.product_name, menu.price,
CASE WHEN members.join_date > sales.order_date THEN 'N'WHEN members.join_date <= sales.order_date THEN 'Y'ELSE 'N' END AS member_status
FROM sales
LEFT JOIN members ON sales.customer_id = members.customer_id
INNER JOIN menu ON sales.product_id = menu.product_id
)

SELECT customer_id,order_date,product_name,price,member_status AS member,
CASE WHEN member_status = 'N' THEN NULL ELSE RANK() OVER (PARTITION BY customer_id, member_status ORDER BY order_date)END AS ranking
FROM customers_data;
```
- Answer:
  - The SQL query starts by creating a Common Table Expression (CTE) named customers_data.
  - Within the CTE, it selects customer_id, order_date, product_name, price, and uses a CASE statement to determine the member_status based on whether the join_date in the members table is greater, equal, or less than the order_date in sales 
    table.
  - The sales table is aliased as sales, the members table as members, and the menu table as menu.
  - The query performs left join between sales and members tables on matching customer_id and inner join between sales and menu tables on matching product_id.
  - For each row, the CASE statement checks the join_date from the members table and compares it to the order_date from the sales table, assigning 'Y' if the join_date is less than or equal to the order_date (customer is a member) and 'N' 
    otherwise (non-member).
  - Next, the main query selects the customer_id, order_date, product_name, price, and member_status from the customers_data CTE.
  - It also uses a CASE statement to calculate the ranking for each customer's orders if they are a member ('Y'). The RANK() function is used with PARTITION BY to partition the ranking within each customer and their membership status and 
    ordered by order_date.
  - If the customer is not a member ('N'), the ranking is set to NULL.
  - Results are presented with the selected columns and ranking for each customer's orders, showing whether they are a member or not and the order of their orders.
 


## Key Insights

- Customer Spending: The total amount spent by each customer at Danny's Diner varies widely. Some customers have spent significantly more than others, indicating potential high-value customers or loyal patrons.
- Customer Visits: The number of days each customer visited the restaurant also varies, showing different patterns of customer engagement. Some customers visit frequently, while others may visit less often.
- First Purchases: Understanding the first items purchased by each customer can help Danny identify popular entry items and potentially attract new customers.
- Most Popular Item: The most purchased item on the menu is valuable information for Danny. He can use this insight to optimize inventory management and capitalize on the popularity of the item.
- Personalized Recommendations: Knowing the most popular item for each customer allows Danny to make personalized menu suggestions, enhancing the dining experience for his customers.
- Customer Loyalty: The data about purchases before and after joining the loyalty program helps Danny evaluate the effectiveness of the loyalty program and its impact on customer behavior.
- Bonus Points for New Members: By offering 2x points to new members during their first week, Danny incentivizes more spending, encouraging members to engage more actively with the program.
- Member Points: The points earned by each member can be used to assess their loyalty and potentially offer targeted rewards and promotions.
- Data Visualization: Creating visualizations based on the data and insights can further aid Danny in understanding trends and making data-driven decisions.
- Customer Segmentation: By analyzing customer spending habits, Danny can segment his customer base and tailor marketing strategies accordingly.
- Expanding Membership: Danny can use the insights from this data to refine his loyalty program and attract new members, leveraging the success of the program.
- Inventory Management: Knowing the most popular and least popular items can help Danny optimize his inventory, reduce wastage, and maximize profits.
- Menu Optimization: Danny can use the data to evaluate the performance of different menu items and consider introducing new dishes based on customer preferences.
- Customer Engagement: Analyzing customer behavior can help Danny understand what keeps customers coming back and help him create more engaging experiences.
- Long-Term Growth: By leveraging data analysis, Danny can make informed decisions that contribute to the long-term growth and success of Danny's Diner.








