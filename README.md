# Olist Brazilian E-Commerce Business Performance Analysis

## Tools
- MySQL, Tableau Public

## Introduction
- Olist E-Commerce Analytics:

## About the dataset
- Data Source: [Brazilian E-Commerce Public Dataset by Olist](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce)
- Data Structure:
<img width="1242" height="670" alt="image" src="https://github.com/user-attachments/assets/d2d4c373-ca4c-48b8-bde6-4b813145ada2" />

## General Performance
### Monthly Order Count, GMV, and Average Order Value
<details>
<summary>View SQL</summary>

```sql
WITH payments AS
(SELECT order_id,SUM(payment_value) AS payment FROM olist_order_payments_dataset GROUP BY order_id)
SELECT
DATE_FORMAT(ood.order_purchase_timestamp, '%Y-%m') AS month,
COUNT(ood.order_id) AS order_count,
ROUND(SUM(p.payment),2) AS gmv,
ROUND(SUM(p.payment)/COUNT(ood.order_id),2) AS avg_order_value
FROM olist_orders_dataset ood
JOIN payments p ON ood.order_id = p.order_id
WHERE ood.order_purchase_timestamp >= '2017-01-01'
AND ood.order_purchase_timestamp < '2018-09-01'
GROUP BY DATE_FORMAT(ood.order_purchase_timestamp, '%Y-%m')
ORDER BY month;
```
</details>
<img width="1485" height="775" alt="14558d11-5c43-43e0-8deb-2889082015e2" src="https://github.com/user-attachments/assets/f485718c-b27e-4c0a-bcd8-37eb39c91d21" />


### Monthly Distribution by Order Value Tier
<details>
<summary>View SQL</summary>
 
```sql
WITH payments AS
(SELECT order_id,SUM(payment_value) AS payment FROM olist_order_payments_dataset GROUP BY order_id)
SELECT
DATE_FORMAT(ood.order_purchase_timestamp, '%Y-%m') AS month,
CASE 
WHEN p.payment < 100 THEN 'low (<100)'
WHEN p.payment < 500 THEN 'mid (100-500)'
ELSE 'high (500+)'
END AS value_tier,
COUNT(ood.order_id) AS order_count
FROM olist_orders_dataset ood
JOIN payments p ON ood.order_id = p.order_id
WHERE ood.order_purchase_timestamp >= '2017-01-01'
AND ood.order_purchase_timestamp < '2018-09-01'
GROUP BY month, value_tier
ORDER BY month, value_tier;
```

</details>
<img width="1461" height="775" alt="62e040dc-e0e5-41e2-a133-d116a5274a9f" src="https://github.com/user-attachments/assets/1d941d86-d073-4eb0-9a54-b3417899126a" />


 
### TOP 10 best-selling products
<details>
<summary>View SQL</summary>
 
```sql
SELECT 
pcnt.product_category_name_english AS product_catagory,
COUNT(*) AS sales_qty,
ROUND(SUM(ooid.price),2) AS product_gmv
FROM olist_order_items_dataset ooid 
INNER JOIN olist_products_dataset opd
ON ooid.product_id = opd.product_id
INNER JOIN product_category_name_translation pcnt
ON opd.product_category_name  = pcnt.product_category_name 
GROUP BY product_catagory ORDER BY product_gmv DESC LIMIT 10;
```

</details>


### TOP 10 seller
<details>
<summary>View SQL</summary>
 
```aql
WITH payment AS
(SELECT order_id,SUM(payment_value) AS order_payment
FROM olist_order_payments_dataset GROUP BY order_id),
top10_sellers AS
(SELECT
ootd.seller_id AS seller,
SUM(p.order_payment) AS gmv
FROM olist_order_items_dataset ootd
JOIN payment p
ON ootd.order_id = p.order_id
GROUP BY ootd.seller_id
ORDER BY gmv DESC LIMIT 10)
SELECT seller, ROUND(gmv,2) AS seller_gmv 
FROM top10_sellers
ORDER BY seller_gmv DESC;
```

</details>
<img width="600" height="290" alt="image" src="https://github.com/user-attachments/assets/e7a705fe-d520-4519-81e8-d7b9a1026554" />

### Top 10 Sellers' Contribution to Total GMV 
<details>
<summary>View SQL</summary>
 
```sql
WITH payment AS
(SELECT order_id,SUM(payment_value) AS order_payment
FROM olist_order_payments_dataset GROUP BY order_id),
top10_sellers AS
(SELECT
ootd.seller_id AS seller,
SUM(p.order_payment) AS gmv
FROM olist_order_items_dataset ootd
JOIN payment p
ON ootd.order_id = p.order_id
GROUP BY ootd.seller_id
ORDER BY gmv DESC LIMIT 10),
T AS
(SELECT 
SUM(gmv) AS top10_seller_gmv,
(SELECT SUM(order_payment) FROM payment)  AS total_gmv 
FROM top10_sellers)
SELECT 'top10 seller' AS catagory,top10_seller_gmv AS gmv FROM T
UNION ALL
SELECT 'all seller' AS catagory,(total_gmv - top10_seller_gmv) AS gmv FROM T;
```
</details>
<img width="100" height="70" alt="image" src="https://github.com/user-attachments/assets/c13b0ee9-20e7-43b3-b7d7-b4052a5edf43" />
<img width="250" height="225" alt="image" src="https://github.com/user-attachments/assets/63667e41-a7ff-4745-884f-ab70f6ea773a" />


### TOP 10 Best Selling Product Categories Rankings by Month
<details>
<summary>View SQL</summary>
 
```sql
WITH T AS
(SELECT order_id,product_id,SUM(price) AS price FROM olist_order_items_dataset ooid GROUP BY order_id,product_id),
T2 AS
(SELECT T.order_id,T.product_id,ood.customer_id,T.price,
DATE_FORMAT(ood.order_purchase_timestamp, '%Y-%m') AS months,
pcnt.product_category_name_english AS product_name
FROM T
INNER JOIN olist_products_dataset opd
ON T.product_id = opd.product_id
INNER JOIN product_category_name_translation pcnt
ON opd.product_category_name = pcnt.product_category_name
INNER JOIN olist_orders_dataset ood 
ON T.order_id = ood.order_id),
T3 AS
(SELECT months,product_name,SUM(price) AS total_price FROM T2 GROUP BY months,product_name),
T4 AS
(SELECT months,product_name,total_price,
DENSE_RANK() OVER (PARTITION BY months ORDER BY total_price DESC) AS rn
FROM T3)
SELECT * FROM T4 WHERE rn <= 10;
```
</details>
<img width="1181" height="863" alt="76ea9daa-bf0f-48d0-92ce-205f6ffa8998" src="https://github.com/user-attachments/assets/5ccb4014-fa23-47f0-8f9c-44af6ec8067f" />
<img width="159" height="89" alt="8b4cfc17-a0b7-4767-97e1-66bccf0eeb46" src="https://github.com/user-attachments/assets/846d2bc5-ac61-4692-8a72-103aee458677" />

## 客户行为类
### Repeat Purchase Rate: 3.12%
<details>
<summary>View SQL</summary>
 
```sql
WITH T AS
(SELECT customer_unique_id,COUNT(customer_unique_id) AS order_count
FROM olist_customers_dataset ocd
GROUP BY customer_unique_id )
SELECT CONCAT(ROUND(COUNT(*)/(SELECT COUNT(DISTINCT customer_unique_id) FROM olist_customers_dataset ocd)*100,2),'%')
AS repeat_purchase_rate 
FROM T WHERE order_count >= 2;
```

<img width="280" height="60" alt="image" src="https://github.com/user-attachments/assets/326c3591-58c1-4b12-9f11-c57bdfe8bc60" />
 </details>
 
### Average Inter-purchase Time: 78 Days
<details>
<summary>View SQL</summary>
 
```sql
WITH T AS
(SELECT ocd.customer_id,ocd.customer_unique_id,ood.order_purchase_timestamp
FROM olist_customers_dataset ocd
INNER JOIN olist_orders_dataset ood
ON ocd.customer_id = ood.customer_id
ORDER BY ood.order_purchase_timestamp),
T2 AS 
(SELECT order_purchase_timestamp,
DATEDIFF(
order_purchase_timestamp,
LAG(order_purchase_timestamp,1) OVER (PARTITION BY customer_unique_id ORDER BY order_purchase_timestamp)) AS prev_order
FROM T)
SELECT ROUND(AVG(prev_order)) AS avg_inter_purchase_time FROM T2;
```
<img width="300" height="60" alt="image" src="https://github.com/user-attachments/assets/0cc776fe-2100-408a-ae21-c04bf3935ab1" />
</details>

### Average Order Value Distribution
<details>
<summary>View SQL</summary>
 
```sql
WITH T AS 
(SELECT order_id,SUM(payment_value) AS order_value
FROM olist_order_payments_dataset oopd
GROUP BY order_id
ORDER BY order_value)
SELECT 
CASE
WHEN order_value >= 500 THEN '500+'
ELSE CONCAT(FLOOR(order_value/50)*50, '-', FLOOR(order_value/50)*50+50)
END AS price_tier,
COUNT(*) AS order_count 
FROM T
GROUP BY price_tier;
```
</details>
<img width="700" height="350" alt="image" src="https://github.com/user-attachments/assets/ae476547-8670-4f2e-90e6-4067639a3880" />

## Regional Analysis
### Order Volume, GMV, and Customer Distribution By State
<details>
<summary>View SQL</summary>
 
```sql
WITH payment AS
(SELECT order_id,SUM(oopd.payment_value) AS order_payment
FROM olist_order_payments_dataset oopd GROUP BY oopd.order_id)
SELECT 
ocd.customer_state AS state,
COUNT(ood.order_id) AS order_Qty,
COUNT(DISTINCT ocd.customer_unique_id) AS cutomer_count,
ROUND(SUM(p.order_payment),2) AS state_gmv
FROM olist_customers_dataset ocd
JOIN olist_orders_dataset ood
ON ocd.customer_id = ood.customer_id
JOIN payment p
ON ood.order_id = p.order_id
GROUP BY state
ORDER BY state_gmv DESC;
```
</details>
<img width="1364" height="780" alt="image" src="https://github.com/user-attachments/assets/356dcd66-8887-432f-bcf1-0bebb30da127" />


### Product Category Revenue by State
<details>
<summary>View SQL</summary>
 
```sql
WITH T AS 
(SELECT ocd.customer_state,ooid.product_id,
SUM(ooid.price) AS total_revenue
FROM olist_order_items_dataset ooid
JOIN olist_orders_dataset ood ON ooid.order_id = ood.order_id
JOIN olist_customers_dataset ocd ON ood.customer_id = ocd.customer_id
GROUP BY ocd.customer_state, ooid.product_id),
T2 AS
(SELECT geolocation_state,
AVG(ogd.geolocation_lat) AS lat,
AVG(ogd.geolocation_lng) AS lng
FROM olist_geolocation_dataset ogd
GROUP BY geolocation_state)
SELECT 
T.customer_state,T2.lat,T2.lng,
pcnt.product_category_name_english AS product_name,
T.total_revenue
FROM T
JOIN olist_products_dataset opd ON T.product_id = opd.product_id
JOIN product_category_name_translation pcnt ON opd.product_category_name = pcnt.product_category_name
JOIN T2 ON T.customer_state  = T2.geolocation_state
ORDER BY T.customer_state,T.total_revenue;
```
</details>
- Data Source: [Product Category Revenue by State]([https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce](https://public.tableau.com/views/Olist_17762903105860/ProductCategoryRevenuebyState))


### Time Latency for Customer Feedback: 3.6 Days
<details>
<summary>View SQL</summary>
 
```sql
WITH T AS
(SELECT oord.order_id,ood.order_delivered_customer_date ,oord.review_answer_timestamp
FROM olist_order_reviews_dataset oord
INNER JOIN olist_orders_dataset ood
ON oord.order_id = ood.order_id
WHERE ood.order_delivered_customer_date IS NOT NULL
AND ood.order_delivered_customer_date != ''),
T2 AS
(SELECT 
order_id,
STR_TO_DATE(order_delivered_customer_date,'%Y-%m-%d %H:%i:%s') AS deliver_date,
STR_TO_DATE(review_answer_timestamp,'%Y/%m/%d %H:%i:%s') AS review_date
FROM T),
T3 AS
(SELECT order_id,
TIMESTAMPDIFF(HOUR, deliver_date, review_date) / 24 AS timediff
FROM T2)
SELECT ROUND(AVG(timediff),1) AS latency FROM T3 WHERE timediff >=0;
```
<img width="186" height="65" alt="image" src="https://github.com/user-attachments/assets/1abe10b1-56da-4db6-8ede-60e5ed039d84" />

</details>

## 物流类
### On Time Delivery Rate: 91.89%
<details>
<summary>View SQL</summary>
 
```sql
WITH T AS
(SELECT order_id,order_purchase_timestamp,order_delivered_customer_date,order_estimated_delivery_date
FROM olist_orders_dataset
WHERE order_delivered_customer_date IS NOT NULL
AND order_delivered_customer_date != ''),
T2 AS
(SELECT order_id,
STR_TO_DATE(order_purchase_timestamp,'%Y-%m-%d %H:%i:%s') AS purchase_time,
STR_TO_DATE(order_delivered_customer_date,'%Y-%m-%d %H:%i:%s') AS deliver_time,
STR_TO_DATE(order_estimated_delivery_date,'%Y-%m-%d %H:%i:%s') AS estimated_delivery
FROM T)
SELECT CONCAT(ROUND(COUNT(*)/(SELECT COUNT(*)FROM T2)*100,2),'%') AS OTD_rate 
FROM T2 WHERE deliver_time <= estimated_delivery;
```

<img width="205" height="65" alt="44a51a43-3324-41d7-b2b4-6deb7410db68" src="https://github.com/user-attachments/assets/9904b799-066b-4790-a308-1f940b2a9f3a" />

</details>


### Average Lead(Delivery) Time: 12.5 Days
<details>
<summary>View SQL</summary>
 
```sql
WITH T AS
(SELECT order_id,order_purchase_timestamp,order_delivered_customer_date,order_estimated_delivery_date
FROM olist_orders_dataset
WHERE order_delivered_customer_date IS NOT NULL
AND order_delivered_customer_date != ''),
T2 AS
(SELECT order_id,
STR_TO_DATE(order_purchase_timestamp,'%Y-%m-%d %H:%i:%s') AS purchase_time,
STR_TO_DATE(order_delivered_customer_date,'%Y-%m-%d %H:%i:%s') AS deliver_time
FROM T),
T3 AS 
(SELECT order_id,deliver_time,purchase_time,
TIMESTAMPDIFF(HOUR,purchase_time,deliver_time)/24 AS lead_ime
FROM T2)
SELECT ROUND(AVG(lead_ime),1) AS avg_lead_ime FROM T3;
```
<img width="225" height="61" alt="image" src="https://github.com/user-attachments/assets/af8b295f-dd72-4092-bde8-020577f8c7b6" />
</details>

### Average Lead Time & Delivery Gap By State
<details>
<summary>View SQL</summary>
 
```sql
WITH T AS
(SELECT customer_id,order_purchase_timestamp,order_delivered_customer_date,order_estimated_delivery_date
FROM olist_orders_dataset
WHERE order_delivered_customer_date IS NOT NULL
AND order_delivered_customer_date != ''),
T2 AS
(SELECT customer_id,
STR_TO_DATE(order_purchase_timestamp,'%Y-%m-%d %H:%i:%s') AS purchase_time,
STR_TO_DATE(order_delivered_customer_date,'%Y-%m-%d %H:%i:%s') AS deliver_time,
STR_TO_DATE(order_estimated_delivery_date,'%Y-%m-%d %H:%i:%s') AS estimated_delivery
FROM T),
T3 AS 
(SELECT customer_id,deliver_time,purchase_time,
DATEDIFF(deliver_time,purchase_time) AS lead_time,
DATEDIFF(estimated_delivery,deliver_time) AS delivery_gap
FROM T2),
T4 AS
(SELECT T3.customer_id,T3.lead_time,T3.delivery_gap,ocd.customer_state
FROM T3 INNER JOIN olist_customers_dataset ocd
ON T3.customer_id = ocd.customer_id)
SELECT
customer_state,
ROUND(AVG(lead_time),2) AS avg_state_lead_time,
ROUND(AVG(delivery_gap),2) AS avg_delivery_gap
FROM T4 
GROUP BY customer_state 
ORDER BY avg_state_lead_time DESC;
```
</details>
<img width="800" height="450" alt="image" src="https://github.com/user-attachments/assets/3284c916-b6a5-4410-b210-1e891186f51a" />


## 评论/满意度类
### Rating Distrabution Ratio
<details>
<summary>View SQL</summary>
 
```sql
SELECT 
review_score,
COUNT(review_score) AS Rating_distribution,
CONCAT(ROUND(COUNT(review_score)/(SELECT COUNT(*) FROM olist_order_reviews_dataset oord)*100,2),'%') AS ration
FROM olist_order_reviews_dataset 
GROUP BY review_score ORDER BY review_score;
```
</details>
<img width="150" height="70" alt="image" src="https://github.com/user-attachments/assets/16c817e5-2bae-4491-bed7-2a598b81a435" />
<img width="350" height="350" alt="image" src="https://github.com/user-attachments/assets/7a5f973f-93bc-45ff-b9b2-d18142c142bf" />

### 物流延误和差评的相关性（图表导出来拖进tableau）
<details>
<summary>View SQL</summary>
 
```sql
WITH T AS
(SELECT order_id,customer_id,order_purchase_timestamp,order_delivered_customer_date,order_estimated_delivery_date
FROM olist_orders_dataset
WHERE order_delivered_customer_date IS NOT NULL
AND order_delivered_customer_date != ''),
T2 AS
(SELECT order_id,customer_id,
STR_TO_DATE(order_purchase_timestamp,'%Y-%m-%d %H:%i:%s') AS purchase_time,
STR_TO_DATE(order_delivered_customer_date,'%Y-%m-%d %H:%i:%s') AS deliver_time,
STR_TO_DATE(order_estimated_delivery_date,'%Y-%m-%d %H:%i:%s') AS estimated_delivery
FROM T),
T3 AS 
(SELECT order_id,customer_id,deliver_time,purchase_time,
DATEDIFF(deliver_time,purchase_time) AS lead_time,
DATEDIFF(estimated_delivery,deliver_time) AS delivery_gap
FROM T2),
T4 AS
(SELECT T3.order_id,T3.customer_id,T3.lead_time,T3.delivery_gap,oord.review_score
FROM T3 INNER JOIN olist_customers_dataset ocd
ON T3.customer_id = ocd.customer_id
INNER JOIN olist_order_reviews_dataset oord
ON T3.order_id = oord.order_id)
SELECT * FROM T4;
```
</details>
<img width="548" height="441" alt="image" src="https://github.com/user-attachments/assets/f33d1b08-b0c0-4eff-9381-c98ae5fe53e7" />


### Review Rate: 41.30%
<details>
<summary>View SQL</summary>
 
```sql
SELECT CONCAT(ROUND(COUNT(review_comment_message)/(SELECT COUNT(*) FROM olist_order_reviews_dataset oord)*100,2),'%')
AS review_rate
FROM olist_order_reviews_dataset
WHERE review_comment_message IS NOT NULL
AND review_comment_message != '';
```

<img width="215" height="61" alt="image" src="https://github.com/user-attachments/assets/1f0768d0-8b14-44bd-9907-e1fa6a94c2ef" />
</details>


### 回复时间对评分的影响(收货和评论间隔时长，以及评分)
<details>
<summary>View SQL</summary>
 
```sql
WITH review_table AS 
(SELECT order_id,review_score, review_answer_timestamp,
ROW_NUMBER() OVER (PARTITION BY order_id ORDER BY review_answer_timestamp DESC) AS rn
FROM olist_order_reviews_dataset),
T AS
(SELECT ood.order_id,
STR_TO_DATE(ood.order_delivered_customer_date,'%Y-%m-%d %H:%i:%s') AS delivery_date,
STR_TO_DATE(rt.review_answer_timestamp,'%Y/%m/%d %H:%i') AS review_date,
rt.review_score
FROM olist_orders_dataset ood 
INNER JOIN review_table rt 
ON ood.order_id = rt.order_id
WHERE ood.order_delivered_customer_date IS NOT NULL 
AND ood.order_delivered_customer_date != ''
AND rn = 1),
T2 AS 
(SELECT order_id,
ROUND(TIMESTAMPDIFF(HOUR,delivery_date,review_date)/24,2) AS time_to_review,
review_score 
FROM T 
WHERE DATEDIFF(review_date,delivery_date) >= 0)
SELECT * FROM T2;
```
</details>
<img width="480" height="320" alt="image" src="https://github.com/user-attachments/assets/0a0940e4-180d-4507-acc5-3e31b9662635" />


## 交叉分析类
### 高价产品是否评分更高（做评分和order价格的对比）
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
</details>
<img width="700" height="350" alt="image" src="https://github.com/user-attachments/assets/a55046d5-9bcd-43d5-8922-f51fcc9b7966" />

### 差评的时间规律 — 差评集中在一周的哪几天、哪个时段提交？情绪是否有周期性？
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
</details>
<img width="598" height="272" alt="image" src="https://github.com/user-attachments/assets/8573f5b1-4178-4b0f-aeb2-34a298bad694" />
<img width="750" height="390" alt="image" src="https://github.com/user-attachments/assets/e32cb163-6ffb-47c1-9d08-3103f84affb5" />


### Customers who left a score but no reviews
<details>
<summary>View SQL</summary>
 
```sql
WITH review_table AS 
(SELECT order_id, review_score, review_comment_title, review_comment_message,
ROW_NUMBER() OVER (PARTITION BY order_id ORDER BY review_answer_timestamp DESC) AS rn
FROM olist_order_reviews_dataset), 
T AS
(SELECT order_id,order_purchase_timestamp,order_delivered_customer_date,order_estimated_delivery_date
FROM olist_orders_dataset
WHERE order_delivered_customer_date IS NOT NULL
AND order_delivered_customer_date != ''),
T2 AS
(SELECT order_id,
STR_TO_DATE(order_purchase_timestamp,'%Y-%m-%d %H:%i:%s') AS purchase_time,
STR_TO_DATE(order_delivered_customer_date,'%Y-%m-%d %H:%i:%s') AS deliver_time
FROM T),
T3 AS 
(SELECT order_id,deliver_time,purchase_time,
DATEDIFF(deliver_time,purchase_time) AS lead_time
FROM T2),
T4 AS
(SELECT order_id,product_id,SUM(price) AS price FROM olist_order_items_dataset ooid GROUP BY order_id,product_id),
T5 AS
(SELECT T4.order_id,T4.product_id,ood.customer_id,T4.price,
DATE_FORMAT(ood.order_purchase_timestamp, '%Y-%m') AS months,
pcnt.product_category_name_english AS product_name
FROM T4
INNER JOIN olist_products_dataset opd
ON T4.product_id = opd.product_id
INNER JOIN product_category_name_translation pcnt
ON opd.product_category_name = pcnt.product_category_name
INNER JOIN olist_orders_dataset ood 
ON T4.order_id = ood.order_id),
T6 AS
(SELECT T3.order_id,T5.product_name,T5.price,T3.lead_time,rt.review_score,rt.review_comment_title,rt.review_comment_message 
FROM T3 INNER JOIN T5 
ON T3.order_id = T5.order_id
INNER JOIN review_table rt
ON T5.order_id = rt.order_id
WHERE rn = 1)
SELECT order_id,product_name,price,lead_time,review_score FROM T6 
WHERE (review_comment_title IS NULL OR review_comment_title = '')
AND (review_comment_message IS NULL OR review_comment_message = ''); 
```
</details>


### Customers who never left a score or review
<details>
<summary>View SQL</summary>
 
```sql
WITH T AS
(SELECT order_id,order_purchase_timestamp,order_delivered_customer_date
FROM olist_orders_dataset
WHERE order_delivered_customer_date IS NOT NULL
AND order_delivered_customer_date != ''),
T2 AS
(SELECT order_id,
STR_TO_DATE(order_purchase_timestamp,'%Y-%m-%d %H:%i:%s') AS purchase_time,
STR_TO_DATE(order_delivered_customer_date,'%Y-%m-%d %H:%i:%s') AS deliver_time
FROM T),
T3 AS 
(SELECT order_id,deliver_time,purchase_time,
DATEDIFF(deliver_time,purchase_time) AS lead_time
FROM T2),
T4 AS
(SELECT order_id,product_id,SUM(price) AS price 
FROM olist_order_items_dataset ooid 
GROUP BY order_id,product_id),
T5 AS
(SELECT T4.order_id,T4.product_id,ood.customer_id,T4.price,
DATE_FORMAT(ood.order_purchase_timestamp, '%Y-%m') AS months,
pcnt.product_category_name_english AS product_name
FROM T4
INNER JOIN olist_products_dataset opd
ON T4.product_id = opd.product_id
INNER JOIN product_category_name_translation pcnt
ON opd.product_category_name = pcnt.product_category_name
INNER JOIN olist_orders_dataset ood 
ON T4.order_id = ood.order_id),
never_reviewed AS 
(SELECT ood.customer_id, ood.order_id
FROM olist_orders_dataset ood
WHERE NOT EXISTS (SELECT 1 FROM olist_order_reviews_dataset rt WHERE ood.order_id = rt.order_id))
SELECT nr.order_id,nr.customer_id,T5.product_name,T5.price,T3.lead_time 
FROM never_reviewed nr
INNER JOIN T3 ON nr.order_id = T3.order_id
INNER JOIN T5 ON nr.order_id = T5.order_id
ORDER BY nr.customer_id;
```
</details>
 
## 结论
- 发现1
- 发现2
