# Olist Brazilian E-Commerce Business Performance Analysis

## Tools
- MySQL, Tableau Public

## Introduction
- Olist E-Commerce Analytics:

## About the dataset
- Data Source: [Brazilian E-Commerce Public Dataset by Olist](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce)
- Data Structure:
<img width="1380" height="745" alt="image" src="https://github.com/user-attachments/assets/d2d4c373-ca4c-48b8-bde6-4b813145ada2" />

## General Performance
### Monthly Order Count, GMV, and Average Order Value
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
### Monthly Distribution by Order Value Tier
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

### TOP 10 best-selling products
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
### TOP 10 seller
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
SELECT seller, ROUND(gmv,2) AS seller_gmv FROM top10_sellers;
```
<img width="480" height="290" alt="image" src="https://github.com/user-attachments/assets/90424168-17e7-44a7-abbe-9b546165e79e" />


### Top 10 Sellers' Contribution to Total GMV
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
ORDER BY gmv DESC LIMIT 10)
SELECT CONCAT(ROUND(SUM(gmv)/(SELECT SUM(order_payment) FROM payment)*100,2),'%') AS CR10 FROM top10_sellers;
```
<img width="170" height="59" alt="image" src="https://github.com/user-attachments/assets/a2f8d179-49c8-4c8f-abe5-f65e2670c22d" />

### TOP 3 Best Selling Product Categories by Month
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
SELECT * FROM T4 WHERE rn <= 3;
```

## 客户行为类
### Repeat Purchase Rate
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
 
### Average Inter-purchase Time
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

### Average Order Value Distribution
```sql
SELECT order_id,SUM(payment_value) AS order_value
FROM olist_order_payments_dataset oopd
GROUP BY order_id
ORDER BY order_value;
```
  
## Regional Analysis
- 各州的订单数量，总销售额&客户总数
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
### TOP 3 Best Selling Product Categories by State
```sql
WITH T AS
(SELECT ooid.order_id,ooid.product_id,ood.customer_id,ocd.customer_state,pcnt.product_category_name_english AS product_name
FROM olist_order_items_dataset ooid
INNER JOIN olist_products_dataset opd
ON ooid.product_id = opd.product_id
INNER JOIN product_category_name_translation pcnt
ON opd.product_category_name = pcnt.product_category_name
INNER JOIN olist_orders_dataset ood 
ON ooid.order_id = ood.order_id
INNER JOIN olist_customers_dataset ocd
ON ood.customer_id = ocd.customer_id),
T2 AS
(SELECT customer_state,product_name,COUNT(*) AS qty
FROM T 
GROUP BY customer_state,product_name),
T3 AS
(SELECT customer_state,product_name,qty,
DENSE_RANK() OVER (PARTITION BY customer_state ORDER BY qty DESC)AS rnk
FROM T2)
SELECT * FROM T3 WHERE rnk <= 3;
```

### Time Latency for Customer Feedback
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
DATEDIFF(review_date, deliver_date) AS timediff
FROM T2)
SELECT ROUND(AVG(timediff),2) AS latency FROM T3 WHERE timediff >=0;
```
<img width="185" height="59" alt="image" src="https://github.com/user-attachments/assets/522078f5-58d6-4455-8ca0-45b55f717c71" />



## 物流类
### On Time Delivery Rate
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

### Average Delivery Time
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
DATEDIFF(deliver_time,purchase_time) AS lead_ime
FROM T2)
SELECT ROUND(AVG(lead_ime),2) AS avg_lead_ime FROM T3;
```
<img width="230" height="63" alt="image" src="https://github.com/user-attachments/assets/04cc9c25-dc1c-49b6-8c08-a5d058ee4a94" />


### Average Lead Time & Delivery Gap By State
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
ORDER BY avg_state_lead_time;
```

## 评论/满意度类
- Rating Distrabution Ratio 比较两级分化
```sql
SELECT 
review_score,
COUNT(review_score) AS Rating_distribution,
CONCAT(ROUND(COUNT(review_score)/(SELECT COUNT(*) FROM olist_order_reviews_dataset oord)*100,2),'%') AS ration
FROM olist_order_reviews_dataset 
GROUP BY review_score ORDER BY review_score;
```
<img width="565" height="165" alt="image" src="https://github.com/user-attachments/assets/c62a5c81-0108-4147-9e92-aa9c05fafc8e" />

### 物流延误和差评的相关性（图表导出来拖进tableau）
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

### 评论回复率
```sql
SELECT CONCAT(ROUND(COUNT(review_comment_message)/(SELECT COUNT(*) FROM olist_order_reviews_dataset oord)*100,2),'%')
AS review_rate
FROM olist_order_reviews_dataset
WHERE review_comment_message IS NOT NULL
AND review_comment_message != '';
```
<img width="213" height="61" alt="image" src="https://github.com/user-attachments/assets/ce1cfdbc-960d-4afb-af23-a604a9d897fb" />

### 回复时间对评分的影响(收货和评论间隔时长，以及评分)
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
SELECT COUNT(*) FROM T2;
```

## 交叉分析类
### 高价产品是否评分更高（做评分和order价格的对比）
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
ORDER BY ooid.price)
SELECT * FROM T;
```

### 卖家评分和销量的关系 avg_review_score 和 total_sales 的对比（注意：先有鸡还是先有蛋？）
```sql
WITH review_table AS 
(SELECT order_id,review_score, review_answer_timestamp,
ROW_NUMBER() OVER (PARTITION BY order_id ORDER BY review_answer_timestamp DESC) AS rn
FROM olist_order_reviews_dataset),
T AS
(SELECT ooid.order_id,
SUM(ooid.price) AS total_price,
rt.review_score
FROM olist_order_items_dataset ooid 
INNER JOIN review_table rt
ON ooid.order_id = rt.order_id
WHERE rn = 1
GROUP BY ooid.order_id,rt.review_score)
SELECT * FROM T ORDER BY total_price;
```

### 差评的时间规律 — 差评集中在一周的哪几天、哪个时段提交？情绪是否有周期性？
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

### Customers who left a score but no reviews
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
### Customers who never left a score or review
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

## 结论
- 发现1
- 发现2
