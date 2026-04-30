# Olist Brazilian E-Commerce Business Performance Analysis

- This is my first end-to-end data analysis project as a self-taught junior analyst transitioning into the field, covering data extraction, transformation, and visualization.

- Currently expanding my skill set with Python as the next step of my data analytics journey.

### Introduction

- Data Source: [Brazilian E-Commerce Public Dataset by Olist](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce)
  
- Tools: MySQL, Tableau 

- The dataset was provided by Olist, a Brazilian e-commerce Store, via Kaggle. The dataset has information of 100k orders from 2016 to 2018 made at multiple marketplaces in Brazil.
  
- This analysis addresses a main question: **Is Olist's growth sustainable?** To answer this, five dimensions are explored: Sales performance, customer behavior, regional distribution, delivery efficiency, and customer feedback. Each dimension reveals a different aspect of the platform's growth dynamics and operational health.

### Data Structure:

<img width="911" height="628" alt="image" src="https://github.com/user-attachments/assets/9ddaa6aa-7708-4571-8ebe-08edf693e22b" />

### Table of Contents and Summary

[1. Sales Performance](#1-sales-performance)

[2. Customer Behavior](#2-customer-behavior)

[3. Regional Analysis](#3-regional-analysis)

[4. Delivery Efficiency](#4-delivery-efficiency)

[5. Customer Feedback and Reviews](#5-customer-feedback-and-reviews)

### Executive Summary

**1. Sales Performance**

Platform GMV and order volume grew steadily over the observed period, although the pace of growth has begun to slow. Seller concentration remains low, which is a healthy sign for the marketplace ecosystem. 

**2. Customer Behavior**

Customer retention is weak, and the average gap between purchases is nearly 80 days. This suggests that growth has been largely driven by new customers rather than repeat buyers, which won't be sustainable in the long run.

**3. Regional Analysis**

São Paulo, Rio de Janeiro, and Minas Gerais account for the majority of orders, leaving much of the country underserved, and those underserved regions are where future growth opportunities lie.

**4. Delivery Efficiency**

Overall delivery performance is reasonable, partly because the platform sets estimated delivery times conservatively to manage customer expectations. However, this approach may damage customer trust due to the lack of transparency. More importantly, delivery quality in remote states fell far behind core markets, and closing this gap would be essential for future regional expansion.

**5. Customer Feedback and Reviews**

Overall ratings are positive, and review participation is high. Ratings are clearly influenced by delivery quality but not by prices. Notably, timing also matters: Scores tend to be higher on Sundays and in the evening, and lower on Mondays and early in the morning. This creates an opportunity to steer customers toward reviewing at times when their sentiment is at its peak. 
 
## 1. Sales Performance
### 1.1 Monthly Order Count, GMV, and Average Order Value
- GMV and order count tracked closely throughout the observed period, both growing steadily from January 2017 and peaking in November, likely driven by Black Friday promotions. The decline in December reflects a post-Black Friday cooldown.
- Average Order Value (AOV) remained largely flat throughout, fluctuating between 145 and 175 BRL with no clear trend. This indicates that GMV growth was driven largely by order volume expansion rather than increased spending per order. 

<details>
<summary>View SQL</summary>

```sql
# Data Filtered:
# Date: Jan 2017 – Sep 2018 (removed outliers with insufficient data volume).
# Status: Excluded 'created', 'canceled', and 'unavailable' (payments unconfirmed/unfulfilled).
# GMV: Product price only. Excludes shipping fee, vouchers, and other payment charges.
WITH orders AS (
SELECT order_id, DATE_FORMAT(order_purchase_timestamp, '%Y-%m') AS months
FROM olist_orders_dataset
WHERE order_purchase_timestamp >= '2017-01-01'
AND order_purchase_timestamp < '2018-09-01'
AND order_status NOT IN ('created', 'canceled', 'unavailable')
)
SELECT o.months,
COUNT(DISTINCT o.order_id) AS order_count,
ROUND(SUM(ooid.price), 2) AS gmv,
ROUND(SUM(ooid.price) / COUNT(DISTINCT o.order_id), 2) AS avg_order_value
FROM orders o
INNER JOIN olist_order_items_dataset ooid ON o.order_id = ooid.order_id
GROUP BY o.months
ORDER BY o.months;
```
<img width="671" height="161" alt="image" src="https://github.com/user-attachments/assets/0c71575c-a8df-47c4-9763-828f935b2750" />

</details>

### 1.2 TOP 10 Best-selling Products
- The Bed/Bath/Table, Health/Beauty, and Sports/Leisure categories dominated in sales volume. However, the main GMV drivers were Bed/Bath/Table, Health/Beauty, and Watches/Gifts.
- This pattern is logically consistent: consumers tend to purchase everyday necessities frequently, while allocating higher budgets for gifts.
<details>
<summary>View SQL</summary>
 
```sql
# Data Filtered:
# Date: Jan 2017 – Sep 2018 (removed outliers with insufficient data volume).
# Status: Excluded 'created', 'canceled', and 'unavailable' (payments unconfirmed/unfulfilled ).
# Sales Quantity: Based on the number of items sold, not the order count, as one order may have multiple identical products.
# Product GMV: Based on product price only, excluding shipping fee.
WITH orders AS (
SELECT order_id,DATE_FORMAT(order_purchase_timestamp, '%Y-%m') AS months
FROM olist_orders_dataset
WHERE order_purchase_timestamp >= '2017-01-01'
AND order_purchase_timestamp < '2018-09-01'
AND order_status NOT IN ('created','canceled', 'unavailable')
)
SELECT 
pcnt.product_category_name_english AS product_category,
COUNT(*) AS sales_qty,
ROUND(SUM(ooid.price),2) AS product_gmv
FROM olist_order_items_dataset ooid
INNER JOIN orders o ON ooid.order_id = o.order_id
INNER JOIN olist_products_dataset opd ON ooid.product_id = opd.product_id
INNER JOIN product_category_name_translation pcnt ON opd.product_category_name = pcnt.product_category_name
GROUP BY product_category 
ORDER BY product_gmv DESC # For the 2nd graph: ORDER BY sales_qty DESC
LIMIT 10;
```
<img width="581" height="165" alt="image" src="https://github.com/user-attachments/assets/35a7cfbf-9646-4c87-850e-b83f382bd22c" />

</details>


### 1.3 TOP 10 Sellers
- The top seller dominates the platform's revenue, surpassing the second-ranked seller by nearly $200k in GMV.
<details>
<summary>View SQL</summary>
 
```sql
# Data Filtered:
# Date: Jan 2017 – Sep 2018 (removed outliers with insufficient data volume).
# Status: Excluded 'created', 'canceled', and 'unavailable' (payments unconfirmed/unfulfilled ).
# GMV: Product price only. Excludes shipping fee, vouchers, and other payment charges.
WITH orders AS (
SELECT order_id,DATE_FORMAT(order_purchase_timestamp, '%Y-%m') AS months
FROM olist_orders_dataset
WHERE order_purchase_timestamp >= '2017-01-01'
AND order_purchase_timestamp < '2018-09-01'
AND order_status NOT IN ('created','canceled', 'unavailable')
)
SELECT 
ooid.seller_id AS seller,
ROUND(SUM(ooid.price), 2) AS seller_gmv
FROM olist_order_items_dataset ooid
INNER JOIN orders o ON ooid.order_id = o.order_id
GROUP BY ooid.seller_id
ORDER BY seller_gmv DESC
LIMIT 10;
```
<img width="471" height="161" alt="image" src="https://github.com/user-attachments/assets/40fba4e9-5960-430d-89e3-fad979598cef" />

</details>

### 1.4 Top 10 Sellers' Contribution to Total GMV 
- Despite the leader's dominance, the total GMV contribution of the Top 10 sellers remains under 20%, indicating a very decentralized and healthy ecosystem where the platform is not overly dependent on a small group of sellers.

<details>
<summary>View SQL</summary>
 
```sql
# Data Filtered:
# Date: Jan 2017 – Sep 2018 (removed outliers with insufficient data volume).
# Status: Excluded 'created', 'canceled', and 'unavailable' (payments unconfirmed/unfulfilled ).
# GMV: Product price only. Excludes shipping fee, vouchers, and other payment charges.
WITH orders AS (
SELECT order_id
FROM olist_orders_dataset
WHERE order_purchase_timestamp >= '2017-01-01'
AND order_purchase_timestamp < '2018-09-01'
AND order_status NOT IN ('created','canceled', 'unavailable')
),
seller_gmv AS (
SELECT ooid.seller_id AS seller,
SUM(ooid.price) AS seller_gmv
FROM olist_order_items_dataset ooid
INNER JOIN orders o ON ooid.order_id = o.order_id
GROUP BY ooid.seller_id
),
top10_seller_gmv AS(
SELECT seller,seller_gmv FROM seller_gmv sg 
ORDER BY seller_gmv DESC LIMIT 10
)
SELECT 'all seller' AS category,SUM(seller_gmv) AS total_gmv FROM seller_gmv
UNION ALL
SELECT 'top10 seller' AS category,SUM(seller_gmv) AS top10_gmv FROM top10_seller_gmv;
```

<img width="355" height="84" alt="image" src="https://github.com/user-attachments/assets/c3082dc1-1db2-481f-a569-0a308ca5ef2b" />

</details>


### 1.5 TOP 10 Best Selling Product Categories Rankings by Month
- The heatmap indicates that most categories appear sporadically within the top 10 rankings, with no recognizable seasonal patterns showing from the map.
- The dominance of Bed/Bath/Table, Health/Beauty, and Sports/Leisure, and Watches/Gifts, is consistent with findings in section 1.2.
  
<details>
<summary>View SQL</summary>
 
```sql
# Data Filtered:
# Date: Jan 2017 – Sep 2018 (removed outliers with insufficient data volume).
# Status: Excluded 'created', 'canceled', and 'unavailable' (payments unconfirmed/unfulfilled ).
# GMV: Product price only. Excludes shipping fee, vouchers, and other payment charges.
WITH orders AS (
SELECT order_id, DATE_FORMAT(order_purchase_timestamp, '%Y-%m') AS months
FROM olist_orders_dataset
WHERE order_purchase_timestamp >= '2017-01-01'
AND order_purchase_timestamp < '2018-09-01'
AND order_status NOT IN ('created', 'canceled', 'unavailable')
),
category_monthly_gmv AS (
SELECT o.months, 
pcnt.product_category_name_english AS product_category,
SUM(ooid.price) AS gmv
FROM olist_order_items_dataset ooid
INNER JOIN orders o ON ooid.order_id = o.order_id
INNER JOIN olist_products_dataset opd ON ooid.product_id = opd.product_id
INNER JOIN product_category_name_translation pcnt ON opd.product_category_name = pcnt.product_category_name
GROUP BY months, product_category
),
gmv_rank AS (
SELECT months, product_category, gmv,
DENSE_RANK() OVER (PARTITION BY months ORDER BY gmv DESC) AS rn
FROM category_monthly_gmv
)
SELECT * FROM gmv_rank WHERE rn <= 10
ORDER BY months, rn;
```
<img width="615" height="160" alt="image" src="https://github.com/user-attachments/assets/cf9747d3-bb2d-429f-b5b8-04d293c953ca" />

</details>



## 2. Customer Behavior
### 2.1 Repeat Purchase Rate: 3.03%
- The repeat purchase rate stands at just **3.03%**, suggesting that the majority of customers make only a single purchase, and indicating this platform doesn't exhibit a healthy loyalty-driven ecosystem.

<details>
<summary>View SQL</summary>
 
```sql
# Data Filtered:
# Date: Jan 2017 – Sep 2018 (removed outliers with insufficient data volume).
# Status: Excluded 'created', 'canceled', and 'unavailable' (payments unconfirmed/unfulfilled ).
WITH customer_orders AS (
SELECT ocd.customer_unique_id,
COUNT(DISTINCT ood.order_id) AS order_count
FROM olist_orders_dataset ood
INNER JOIN olist_customers_dataset ocd ON ood.customer_id = ocd.customer_id
WHERE ood.order_purchase_timestamp >= '2017-01-01'
AND ood.order_purchase_timestamp < '2018-09-01'
AND ood.order_status NOT IN ('created', 'canceled', 'unavailable')
GROUP BY ocd.customer_unique_id
)
SELECT 
CONCAT(ROUND(COUNT(CASE WHEN order_count >= 2 THEN 1 END) / COUNT(*) * 100, 2), '%')
AS repeat_purchase_rate
FROM customer_orders;
```
<img width="279" height="60" alt="image" src="https://github.com/user-attachments/assets/7ee53560-f1db-431e-8e40-0c5e130d7be0" />

 </details>
 
### 2.2 Average Inter-purchase Time: 78 days
- The average inter-purchase interval is approximately **78 days**, which indicates a low purchase frequency and reinforces the previous conclusion of weak customer loyalty.
<details>
<summary>View SQL</summary>
 
```sql
# Data Filtered:
# Date: Jan 2017 – Sep 2018 (removed outliers with insufficient data volume).
# Status: Excluded 'created', 'canceled', and 'unavailable' (payments unconfirmed/unfulfilled ).
WITH orders AS (
SELECT ocd.customer_unique_id,ood.order_purchase_timestamp
FROM olist_customers_dataset ocd
INNER JOIN olist_orders_dataset ood ON ocd.customer_id = ood.customer_id
WHERE ood.order_purchase_timestamp >= '2017-01-01'
AND ood.order_purchase_timestamp < '2018-09-01'
AND ood.order_status NOT IN ('created', 'canceled', 'unavailable')
),
intervals AS (
SELECT customer_unique_id,
DATEDIFF(
order_purchase_timestamp,
LAG(order_purchase_timestamp) OVER (PARTITION BY customer_unique_id ORDER BY order_purchase_timestamp)) AS order_interval
FROM orders
)
SELECT ROUND(AVG(order_interval)) AS avg_interval
FROM intervals
WHERE order_interval IS NOT NULL;
```
<img width="225" height="61" alt="image" src="https://github.com/user-attachments/assets/87918ee5-a6c8-451d-8096-cb8405691315" />

</details>

### 2.3 Average Order Value Distribution
- Orders in the 50–100 BRL price range dominate in sales volume, with the majority of orders falling below 200 BRL.
- For orders exceeding 100 BRL, a clear inverse relationship is observed between order value and volume: The higher the price, the fewer the orders.
- This distribution indicates that Olist's customer base is primarily price-sensitive and driven by low-value transactions.

<details>
<summary>View SQL</summary>
 
```sql
# Data Filtered:
# Date: Jan 2017 – Sep 2018 (removed outliers with insufficient data volume).
# Status: Excluded 'created', 'canceled', and 'unavailable' (payments unconfirmed/unfulfilled).
# GMV: Product price only. Excludes shipping fee, vouchers, and other payment charges.
WITH orders AS 
(SELECT order_id,DATE_FORMAT(order_purchase_timestamp, '%Y-%m') AS months
FROM olist_orders_dataset
WHERE order_purchase_timestamp >= '2017-01-01'
AND order_purchase_timestamp < '2018-09-01'
AND order_status NOT IN ('created','canceled', 'unavailable')
),
gmv AS (
SELECT o.order_id, SUM(ooid.price) AS order_value
FROM olist_order_items_dataset ooid 
INNER JOIN orders o ON ooid.order_id = o.order_id 
GROUP BY o.order_id
)
SELECT 
CASE
WHEN order_value >= 500 THEN '500+'
ELSE CONCAT(FLOOR(order_value/50)*50, '-', FLOOR(order_value/50)*50+50)
END AS price_tier,
COUNT(*) AS order_count 
FROM gmv
GROUP BY price_tier
ORDER BY MIN(order_value);
```
<img width="365" height="164" alt="image" src="https://github.com/user-attachments/assets/2f25e014-a2f3-4618-95b9-50fb4a72f291" />

</details>


## 3. Regional Analysis
### 3.1 Order Volume, GMV, and Customer Distribution By State
- SP (São Paulo), RJ (Rio de Janeiro), and MG (Minas Gerais) dominated Olist’s market, which reflected Brazil’s economic and logistical landscape in 2018, where SP, RJ, and MG dominated purchasing power.
- While the Brazilian e-commerce sector showed consistent growth before and after 2018, the country's e-commerce didn't experience its major boom until 2020. Thus, Olist’s 2018 concentration in the 'Golden Triangle' (SP-RJ-MG) is a logical outcome of that era's market limitations.
<details>
<summary>View SQL</summary>
 
```sql
# Data Filtered:
# Date: Jan 2017 – Sep 2018 (removed outliers with insufficient data volume).
# Status: Excluded 'created', 'canceled', and 'unavailable' (payments unconfirmed/unfulfilled ).
# GMV: Product price only. Excludes shipping fee, vouchers, and other payment charges.
WITH orders AS 
(SELECT order_id,customer_id,DATE_FORMAT(order_purchase_timestamp, '%Y-%m') AS months
FROM olist_orders_dataset
WHERE order_purchase_timestamp >= '2017-01-01'
AND order_purchase_timestamp < '2018-09-01'
AND order_status NOT IN ('created','canceled', 'unavailable')
),
gmv AS (
SELECT o.order_id, SUM(ooid.price) AS order_value
FROM olist_order_items_dataset ooid 
INNER JOIN orders o ON ooid.order_id = o.order_id 
GROUP BY o.order_id
)
SELECT 
ocd.customer_state AS state,
COUNT(o.order_id) AS order_Qty,
COUNT(DISTINCT ocd.customer_unique_id) AS cutomer_count,
ROUND(SUM(g.order_value),2) AS state_gmv
FROM olist_customers_dataset ocd
JOIN orders o ON ocd.customer_id = o.customer_id
JOIN gmv g ON o.order_id = g.order_id
GROUP BY state
ORDER BY state_gmv DESC;
```
<img width="670" height="165" alt="image" src="https://github.com/user-attachments/assets/c8a726fe-c877-43c3-9e8b-6bdca77be41c" />

</details>


### 3.2 Product Category Revenue by State
- The map below visualises total product revenue by state across Brazil. As Tableau was unable to recognise Brazil's state abbreviations, I defined state locations based on their latitude and longitude. This allowed map rendering, while accuracy may vary for large states.
- Readers are encouraged to interact with the visualisation directly: product categories can be filtered using the panel on the right, and selecting an individual state will display its corresponding total revenue.
<details>
<summary>View SQL</summary>
 
```sql
# Data Filtered:
# Date: Jan 2017 – Sep 2018 (removed outliers with insufficient data volume).
# Status: Excluded 'created', 'canceled', and 'unavailable' (payments unconfirmed/unfulfilled ).
# GMV: Product price only. Excludes shipping fee, vouchers, and other payment charges.
WITH orders AS 
(SELECT order_id,customer_id,DATE_FORMAT(order_purchase_timestamp, '%Y-%m') AS months
FROM olist_orders_dataset
WHERE order_purchase_timestamp >= '2017-01-01'
AND order_purchase_timestamp < '2018-09-01'
AND order_status NOT IN ('created','canceled', 'unavailable')
),
gmv AS 
(SELECT ocd.customer_state,ooid.product_id,
SUM(ooid.price) AS total_revenue
FROM olist_order_items_dataset ooid
JOIN orders o ON ooid.order_id = o.order_id
JOIN olist_customers_dataset ocd ON o.customer_id = ocd.customer_id
GROUP BY ocd.customer_state, ooid.product_id),
location AS
(SELECT geolocation_state,
AVG(ogd.geolocation_lat) AS lat,
AVG(ogd.geolocation_lng) AS lng
FROM olist_geolocation_dataset ogd
GROUP BY geolocation_state)
SELECT 
g.customer_state,l.lat,l.lng,
pcnt.product_category_name_english AS product_name,
g.total_revenue
FROM gmv g
JOIN olist_products_dataset opd ON g.product_id = opd.product_id
JOIN product_category_name_translation pcnt ON opd.product_category_name = pcnt.product_category_name
JOIN location l ON g.customer_state  = l.geolocation_state
ORDER BY g.customer_state,g.total_revenue DESC;
```
<img width="840" height="163" alt="image" src="https://github.com/user-attachments/assets/f7af55e5-61d2-428e-b39f-583bca9ad2a4" />

</details>

- Interactive map: [Product Category Revenue by State]( )

## 4. Delivery Efficiency
- Lead time = actual delivery time
- Delivery gap = estimated delivery time - actual delivery time

### 4.1 On Time Delivery Rate: 91.87%
- Olist recorded an on-time delivery rate of 91.89%, which appears healthy. However, it may indicate that the estimated delivery time is conservatively set to manage customer expectations.
<details>
<summary>View SQL</summary>
 
```sql
# Data Filtered:
# Date: Jan 2017 – Sep 2018 (removed outliers with insufficient data volume).
# Status: Excluded 'created', 'canceled', and 'unavailable' (payments unconfirmed/unfulfilled ).
WITH orders AS 
(SELECT order_id,
STR_TO_DATE(NULLIF(TRIM(order_purchase_timestamp), ''), '%Y-%m-%d %H:%i:%s') AS purchase_time,
STR_TO_DATE(NULLIF(TRIM(order_delivered_customer_date), ''), '%Y-%m-%d %H:%i:%s') AS deliver_time,
STR_TO_DATE(NULLIF(TRIM(order_estimated_delivery_date), ''), '%Y-%m-%d %H:%i:%s') AS estimated_delivery
FROM olist_orders_dataset
WHERE order_purchase_timestamp >= '2017-01-01'
AND order_purchase_timestamp < '2018-09-01'
AND order_status NOT IN ('created','canceled', 'unavailable')
AND order_purchase_timestamp  IS NOT NULL AND order_purchase_timestamp  != ''
AND order_delivered_customer_date  IS NOT NULL AND order_delivered_customer_date  != ''
AND order_estimated_delivery_date  IS NOT NULL AND order_estimated_delivery_date  != ''
)
SELECT CONCAT(ROUND(COUNT(*)/(SELECT COUNT(*)FROM orders)*100,2),'%') AS OTD_rate 
FROM orders WHERE deliver_time <= estimated_delivery;
```
<img width="201" height="59" alt="image" src="https://github.com/user-attachments/assets/102a2959-f0c2-457f-abdd-bce5f4143e8a" />

</details>


### 4.2 Average Lead(Delivery) Time: 12.5 Days
- The national average lead time of 12.5 days provides limited insight, as logistical performance can vary greatly across states, therefore it will be further examined in the following section.
<details>
<summary>View SQL</summary>
 
```sql
# Data Filtered:
# Date: Jan 2017 – Sep 2018 (removed outliers with insufficient data volume).
# Status: Excluded 'created', 'canceled', and 'unavailable' (payments unconfirmed/unfulfilled ).
WITH orders AS 
(SELECT order_id,
STR_TO_DATE(order_purchase_timestamp,'%Y-%m-%d %H:%i:%s') AS purchase_time,
STR_TO_DATE(order_delivered_customer_date,'%Y-%m-%d %H:%i:%s') AS deliver_time
FROM olist_orders_dataset
WHERE order_purchase_timestamp >= '2017-01-01'
AND order_purchase_timestamp < '2018-09-01'
AND order_status NOT IN ('created','canceled', 'unavailable')
AND order_purchase_timestamp  IS NOT NULL AND order_purchase_timestamp  != ''
AND order_delivered_customer_date IS NOT NULL AND order_delivered_customer_date  != ''
)
SELECT ROUND(AVG(TIMESTAMPDIFF(HOUR,purchase_time,deliver_time)/24),1) AS avg_lead_time FROM orders;
```
<img width="225" height="61" alt="image" src="https://github.com/user-attachments/assets/af8b295f-dd72-4092-bde8-020577f8c7b6" />
</details>

### 4.3 Average Lead Time & Delivery Gap By State
- The gap between estimated and actual delivery time was positive among all states, indicating that the estimated delivery time(EDT) is conservatively set to mitigate the risk of delayed delivery. It's a pragmatic approach to adapt the logistical environment in 2018, but it means the 91.89% on-time rate is largely due to inflation, which may reduce transparency for customers when evaluating delivery expectations.
- When it comes to lead time, the dispersed data points indicate a highly fragmented logistical landscape, suggesting this country's logistical infrastructure varies considerably.
- In addition, the northern states of RO, AC, AM, AP, and RR showed disproportionately large delivery gaps, suggesting these regions have structural logistical uncertainty.
- Ultimately, Olist's delivery reliability was heavily dictated by national geography.

<details>
<summary>View SQL</summary>
 
```sql

# Data Filtered:
# Date: Jan 2017 – Sep 2018 (removed outliers with insufficient data volume).
# Status: Excluded 'created', 'canceled', and 'unavailable' (payments unconfirmed/unfulfilled ).
WITH orders AS 
(SELECT order_id, customer_id,
STR_TO_DATE(NULLIF(TRIM(order_purchase_timestamp), ''), '%Y-%m-%d %H:%i:%s') AS purchase_time,
STR_TO_DATE(NULLIF(TRIM(order_delivered_customer_date), ''), '%Y-%m-%d %H:%i:%s') AS deliver_time,
STR_TO_DATE(NULLIF(TRIM(order_estimated_delivery_date), ''), '%Y-%m-%d %H:%i:%s') AS estimated_delivery
FROM olist_orders_dataset
WHERE order_purchase_timestamp >= '2017-01-01'
AND order_purchase_timestamp < '2018-09-01'
AND order_status NOT IN ('created','canceled', 'unavailable')
AND order_purchase_timestamp  IS NOT NULL AND order_purchase_timestamp  != ''
AND order_delivered_customer_date  IS NOT NULL AND order_delivered_customer_date  != ''
AND order_estimated_delivery_date  IS NOT NULL AND order_estimated_delivery_date  != ''
),
time AS
(SELECT order_id, o.customer_id,
DATEDIFF(deliver_time,purchase_time) AS lead_time,
DATEDIFF(estimated_delivery,deliver_time) AS delivery_gap,
ocd.customer_state
FROM orders o INNER JOIN olist_customers_dataset ocd
ON o.customer_id = ocd.customer_id)
SELECT
customer_state,
ROUND(AVG(lead_time),2) AS avg_state_lead_time,
ROUND(AVG(delivery_gap),2) AS avg_delivery_gap
FROM time
GROUP BY customer_state 
ORDER BY avg_state_lead_time DESC;
```
<img width="661" height="161" alt="image" src="https://github.com/user-attachments/assets/0e704c4c-d964-4601-b71a-1208e3f86f5d" />

</details>


## 5. Customer Feedback and Reviews
### 5.1 Rating Distribution
- The review scores showed a classic J-shaped distribution, where customers are more likely to leave feedback at the extremes of their experience. But notably, the customer satisfaction rate on the platform is relatively healthy, with 5-star and 4-star ratings representing 77% of the feedback, considerably larger than 1-star (11.51%), 2-star (3.18%), and 3-star (8.24%).
<details>
<summary>View SQL</summary>
 
```sql
# Data Filtered:
# Date: Jan 2017 – Sep 2018 (removed outliers with insufficient data volume).
# Status: Excluded 'created', 'canceled', and 'unavailable' (payments unconfirmed/unfulfilled ).
WITH orders AS
(SELECT order_id
FROM olist_orders_dataset
WHERE order_purchase_timestamp >= '2017-01-01'
AND order_purchase_timestamp < '2018-09-01'
AND order_status NOT IN ('created','canceled', 'unavailable')
),
review_counts AS (
SELECT oord.review_score, COUNT(*) AS rating_counts
FROM olist_order_reviews_dataset oord
INNER JOIN orders o ON o.order_id = oord.order_id
GROUP BY oord.review_score
)
SELECT review_score, rating_counts
FROM review_counts
ORDER BY review_score;
```
<img width="565" height="160" alt="image" src="https://github.com/user-attachments/assets/9e75cd29-423f-444a-8422-a0f2742c3a66" />

</details>
<img width="150" height="70" alt="image" src="https://github.com/user-attachments/assets/16c817e5-2bae-4491-bed7-2a598b81a435" />
<img width="350" height="350" alt="image" src="https://github.com/user-attachments/assets/7a5f973f-93bc-45ff-b9b2-d18142c142bf" />

### 5.2 Correlation: Lead Time vs. Rating
- The correlation between lead time and rating is quite intuitive: The quicker the orders arrive, the higher the ratings might be.
- Alternatively, the delivery gap also showed a similar correlation: The bigger the gap between the estimated and actual delivery date, the higher the customer satisfaction. 
<details>
<summary>View SQL</summary>
 
```sql
# Data Filtered:
# Date: Jan 2017 – Sep 2018 (removed outliers with insufficient data volume).
# Status: Excluded 'created', 'canceled', and 'unavailable' (payments unconfirmed/unfulfilled ).
WITH orders AS
(SELECT order_id, customer_id,
STR_TO_DATE(NULLIF(TRIM(order_purchase_timestamp), ''), '%Y-%m-%d %H:%i:%s') AS purchase_time,
STR_TO_DATE(NULLIF(TRIM(order_delivered_customer_date), ''), '%Y-%m-%d %H:%i:%s') AS deliver_time,
STR_TO_DATE(NULLIF(TRIM(order_estimated_delivery_date), ''), '%Y-%m-%d %H:%i:%s') AS estimated_delivery
FROM olist_orders_dataset
WHERE order_purchase_timestamp >= '2017-01-01'
AND order_purchase_timestamp < '2018-09-01'
AND order_status NOT IN ('created','canceled', 'unavailable')
AND order_purchase_timestamp  IS NOT NULL AND order_purchase_timestamp  != ''
AND order_delivered_customer_date  IS NOT NULL AND order_delivered_customer_date  != ''
AND order_estimated_delivery_date  IS NOT NULL AND order_estimated_delivery_date  != ''
),
review_counts AS (
SELECT o.order_id, oord.review_score
FROM olist_order_reviews_dataset oord
INNER JOIN orders o ON o.order_id = oord.order_id
)
SELECT o.order_id, o.customer_id, r.review_score,
DATEDIFF(o.deliver_time,o.purchase_time) AS lead_time,
DATEDIFF(o.estimated_delivery,o.deliver_time) AS delivery_gap
FROM orders o
INNER JOIN review_counts r ON o.order_id = r.order_id
ORDER BY review_score;
```


</details>
<img width="548" height="441" alt="image" src="https://github.com/user-attachments/assets/f33d1b08-b0c0-4eff-9381-c98ae5fe53e7" />


### 5.3 Correlation: Product Price vs. Rating
- The line chart reveals no clear correlation between the product price and review score. Ratings remained largely stable across all price tiers, fluctuating narrowly between 3.90 and 4.16.
- A notable spike at the 300–350 BRL tier and a dip at 350–400 BRL show slight variation, though these fluctuations are likely due to the diverse product mix within each tier rather than price itself.
- Overall, product price appears to have minimal influence on customer satisfaction, implying that other factors(such as delivery performance and product quality) are stronger affectors.
<details>
<summary>View SQL</summary>
 
```sql
WITH review_table AS 
(SELECT order_id,review_score, review_answer_timestamp,
ROW_NUMBER() OVER (PARTITION BY order_id ORDER BY review_answer_timestamp DESC) AS rn
FROM olist_order_reviews_dataset),
T AS
(SELECT ooid.order_id,ooid.price,rt.review_score
FROM olist_order_items_dataset ooid 
INNER JOIN review_table rt
ON ooid.order_id = rt.order_id
WHERE rn = 1
ORDER BY ooid.price),
T2 AS
(SELECT order_id,
CASE
WHEN price >= 500 THEN '500+'
ELSE CONCAT(FLOOR(price/50)*50, '-', FLOOR(price/50)*50+50)
END AS price_tier,
review_score
FROM T)
SELECT * FROM T2;
```
<img width="640" height="159" alt="image" src="https://github.com/user-attachments/assets/3a9d0bb7-515c-4e9c-b293-dbd15c104a40" />

</details>
<img width="700" height="350" alt="image" src="https://github.com/user-attachments/assets/a55046d5-9bcd-43d5-8922-f51fcc9b7966" />

### 5.4 Rating Trends by Time of Day & Day of Week
- The data suggests that customer sentiment is closely linked to specific time cycles:
- Weekly Trend: The average ratings reach the lowest point on Monday (~4.04), potentially reflecting a "Monday Blues" effect, then the ratings steadily improve throughout the week, peaking on Thursday (~4.11), which is counter-intuitive, as it’s higher than Friday and Saturday.
- Daily Trend: Early morning (4–7 AM) shows the lowest ratings (3.67 – 3.78), with the lowest dip occurring at 5 am (~3.7). Starting from here, ratings climb consistently as the day progresses, peaking late at night (23:00) before dropping again.
- These patterns reveal that ratings are not just a measure of service and product quality, but are also influenced by the timing of the review submission. Customers are most generous when relaxing late at night and most critical during early morning or the start of the work week.

<details>
<summary>View SQL</summary>
 
```sql
WITH T AS 
(SELECT review_id,
STR_TO_DATE(review_answer_timestamp,'%Y/%m/%d %H:%i') AS review_time,
review_score
FROM olist_order_reviews_dataset
WHERE review_answer_timestamp IS NOT NULL
AND review_answer_timestamp != ''),
T2 AS
(SELECT review_id,review_time,
DAYOFWEEK(review_time) AS review_day_of_week,
HOUR(review_time) AS review_hour,
review_score
FROM T)
SELECT review_day_of_week,COUNT(*),ROUND(AVG(review_score),2) AS avg_score FROM T2 GROUP BY review_day_of_week ORDER BY review_day_of_week;
SELECT review_hour,COUNT(*) AS review_count,ROUND(AVG(review_score),2) AS avg_score FROM T2 GROUP BY review_hour ORDER BY review_hour;
```
<img width="581" height="210" alt="image" src="https://github.com/user-attachments/assets/7cd7ed98-6882-4f8b-aa3a-63d5e4ba47bd" />
<img width="553" height="166" alt="image" src="https://github.com/user-attachments/assets/f77cc3d4-7a18-4167-baaf-4a0f916fdb31" />

</details>
<img width="126" height="54" alt="image" src="https://github.com/user-attachments/assets/b42f268c-484a-41ce-b533-246bc4009aac" />
<img width="612" height="287" alt="image" src="https://github.com/user-attachments/assets/632e698c-f392-434e-b20c-13a72cd96c8e" />
<img width="863" height="356" alt="image" src="https://github.com/user-attachments/assets/5248bd9e-baa2-4a57-a7b9-845f2d1f62d2" />

### 5.5 Review Participation Distribution
#### Written Review Orders， Score-Only Orders, and No Review Orders Distribution
- Of all orders, 42.95% received a written review (either a comment title, message, or both), while 56.52% of the orders only received a rating score and no written comment. A small fraction of 768 orders (0.77%) have no review record at all.

<details>
<summary>View SQL</summary>
 
```sql
WITH reviews AS 
(SELECT order_id, review_score, review_comment_title, review_comment_message,
ROW_NUMBER() OVER (PARTITION BY order_id ORDER BY review_answer_timestamp DESC) AS rn
FROM olist_order_reviews_dataset)
SELECT
SUM(CASE WHEN r.order_id IS NULL THEN 1 ELSE 0 END) AS no_review,
SUM(CASE WHEN r.order_id IS NOT NULL 
AND (r.review_comment_title IS NULL OR r.review_comment_title = '')
AND (r.review_comment_message IS NULL OR r.review_comment_message = '') 
THEN 1 ELSE 0 END) AS score_onlyt,
SUM(CASE WHEN (r.review_comment_title IS NOT NULL AND r.review_comment_title != '')
OR (r.review_comment_message IS NOT NULL AND r.review_comment_message != '') 
THEN 1 ELSE 0 END) AS full_review
FROM olist_orders_dataset o
LEFT JOIN reviews r ON o.order_id = r.order_id AND r.rn = 1;
```
<img width="529" height="64" alt="image" src="https://github.com/user-attachments/assets/46863da3-6b97-47f5-8529-8bc7cc7316dc" />

</details>

<img width="791" height="157" alt="image" src="https://github.com/user-attachments/assets/9cb38e96-e325-417e-9b00-9c4e9fc1977a" />

#### Average Review Scores: Written Review Orders vs. Score-Only Orders
- Customers who left a score without a written review gave an average rating of **4.38**, notably higher than the **3.70** average for those who wrote a review. This suggests that dissatisfied customers are more likely to leave a written review, while satisfied customers tend to leave a score only: The louder the customer, the lower the score."
  
<details>
<summary>View SQL</summary>
 
```sql
WITH reviews AS
(SELECT order_id, review_score, review_comment_title, review_comment_message,
ROW_NUMBER() OVER (PARTITION BY order_id ORDER BY review_answer_timestamp DESC) AS rn
FROM olist_order_reviews_dataset)
SELECT
ROUND(AVG(CASE WHEN r.order_id IS NOT NULL
AND (r.review_comment_title IS NULL OR r.review_comment_title = '')
AND (r.review_comment_message IS NULL OR r.review_comment_message = '')
THEN r.review_score END), 2) AS score_only_avg,
ROUND(AVG(CASE WHEN (r.review_comment_title IS NOT NULL AND r.review_comment_title != '')
OR (r.review_comment_message IS NOT NULL AND r.review_comment_message != '')
THEN r.review_score END), 2) AS full_review_avg
FROM olist_orders_dataset o
LEFT JOIN reviews r ON o.order_id = r.order_id AND r.rn = 1;
```

<img width="429" height="61" alt="image" src="https://github.com/user-attachments/assets/a97a6a71-0992-4aac-a65f-24d9ab18ebed" />

</details>
<img width="343" height="206" alt="image" src="https://github.com/user-attachments/assets/ab8073ff-4313-48cf-866b-f27bd383c2f4" />


