# 项目标题

## Tools
### MySQL, Tableau Public

## Introduction
### Olist E-Commerce Analytics: 

## General Performance
### Monthly GMV
```sql
SELECT
DATE_FORMAT(o.order_purchase_timestamp, '%Y-%m') AS month,
COUNT(o.order_id) AS order_count,
ROUND(SUM(p.payment_value),2) AS gmv
FROM olist_orders_dataset o
JOIN olist_order_payments_dataset p ON o.order_id = p.order_id
WHERE o.order_purchase_timestamp >= '2017-01-01'
AND o.order_purchase_timestamp < '2018-09-01'
GROUP BY DATE_FORMAT(o.order_purchase_timestamp, '%Y-%m')
ORDER BY month;
```
### TOP 10 best-selling product categories and GMV
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
GROUP BY product_catagory ORDER BY sales_qty DESC LIMIT 10;
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

### Top 10 Seller GVM
```sql
SELECT ROUND(SUM(gmv),2) AS seller_gmv FROM top10_sellers;
```
<img width="210" height="60" alt="image" src="https://github.com/user-attachments/assets/0d20af53-4fd2-4dbe-838f-40938c4fb274" />

### Total GVM: 16008872.12
```sql
SELECT ROUND(SUM(order_payment),2) AS total_gmv FROM payment;
```
<img width="206" height="61" alt="image" src="https://github.com/user-attachments/assets/bbc050d2-7ac7-4081-8fb0-6d7ea1cd4238" />

### Top 10 Sellers' Contribution to Total GMV
```sql
SELECT CONCAT(ROUND(SUM(gmv)/(SELECT SUM(order_payment) FROM payment)*100,2),'%') AS CR10 FROM top10_sellers;
```
<img width="170" height="59" alt="image" src="https://github.com/user-attachments/assets/a2f8d179-49c8-4c8f-abe5-f65e2670c22d" />

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
- 各州排名前三的产品
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

### 回复时间对评分的影响

## 交叉分析类
### 高价产品是否评分更高（做评分和order价格的对比）

### 节假日前后购买行为和评分变化- 只看黑五 Black Friday

### 卖家评分和销量的关系 avg_review_score 和 total_sales 的对比（注意：先有鸡还是先有蛋？）

## 结论
- 发现1
- 发现2
