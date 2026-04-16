# Olist Brazilian E-Commerce Business Performance Analysis

## Tools
- MySQL, Tableau Public

## Introduction
- Olist E-Commerce Analytics:

## About the dataset
- Data Source: [Brazilian E-Commerce Public Dataset by Olist](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce)
- Data Structure:
<img width="911" height="628" alt="image" src="https://github.com/user-attachments/assets/9ddaa6aa-7708-4571-8ebe-08edf693e22b" />


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
<img width="668" height="165" alt="image" src="https://github.com/user-attachments/assets/795d8f5f-53ff-4584-af05-b256b591331a" />

</details>
<img width="117" height="49" alt="image" src="https://github.com/user-attachments/assets/92db767d-70ab-481e-8791-5edcb50ab672" />

<img width="837" height="517" alt="image" src="https://github.com/user-attachments/assets/07f6572f-0e67-419a-83ba-a62fb98b9820" />

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
<img width="504" height="185" alt="image" src="https://github.com/user-attachments/assets/447323aa-9dc9-4bb2-85dc-6fe824f5051b" />

</details>
<img width="117" height="52" alt="image" src="https://github.com/user-attachments/assets/8355a408-7c39-4ac4-8855-1b7b1eb26ca4" />
<img width="878" height="462" alt="image" src="https://github.com/user-attachments/assets/5fef912c-4f66-4eba-8673-8a60a154ad5a" />
 
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
<img width="579" height="156" alt="image" src="https://github.com/user-attachments/assets/22f8e7a3-d705-4102-8aa1-90b2b7b3fac3" />

</details>
<img width="619" height="490" alt="image" src="https://github.com/user-attachments/assets/f69df1f6-9ed6-41bd-b184-49dc9494354e" />


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

<img width="480" height="156" alt="image" src="https://github.com/user-attachments/assets/5812e081-c072-4d46-ba27-4cdd456c96b8" />

</details>
<img width="588" height="283" alt="image" src="https://github.com/user-attachments/assets/8bdf35f4-d9e0-41c9-aa68-57c34ef534fd" />

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

<img width="356" height="85" alt="image" src="https://github.com/user-attachments/assets/2850ec26-64a4-4201-8a11-082325614b16" />

</details>
<img width="90" height="23" alt="image" src="https://github.com/user-attachments/assets/8e4704e6-a65c-4566-8e83-0163142bf973" />
<img width="364" height="361" alt="image" src="https://github.com/user-attachments/assets/7832e289-2fc8-44f4-a29e-482cc7c02ca5" />

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
<img width="635" height="165" alt="image" src="https://github.com/user-attachments/assets/bf26c377-b7a1-48d1-80b1-0138b0685fdc" />

</details>

- Interactive Graph:[Product Category Revenue by State](https://public.tableau.com/views/Olist_17762903105860/ProductCategoryRevenuebyState)

<img width="131" height="52" alt="image" src="https://github.com/user-attachments/assets/dc29261b-3250-46d6-8e60-fa2b2b7cbf9b" />
<img width="843" height="540" alt="image" src="https://github.com/user-attachments/assets/7c970873-8b64-47be-90dd-a1a6579cad09" />


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
<img width="365" height="160" alt="image" src="https://github.com/user-attachments/assets/3e66617f-11db-4013-9c93-7e18aaaa87e5" />

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
<img width="665" height="155" alt="image" src="https://github.com/user-attachments/assets/f2175ea4-6bba-4365-b327-a55bac38f79f" />

</details>
<img width="157" height="48" alt="image" src="https://github.com/user-attachments/assets/7045f92a-572a-47b8-acdb-d5964bae64fb" />
<img width="1064" height="576" alt="image" src="https://github.com/user-attachments/assets/9bda85b0-aa37-4381-99c7-2ad5cc459321" />

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
<img width="835" height="160" alt="image" src="https://github.com/user-attachments/assets/f48ddc1f-814e-42ce-a7c8-5d5b1fad1015" />

</details>

- Interactive map: [Product Category Revenue by State](https://public.tableau.com/views/Olist_17762903105860/ProductCategoryRevenuebyState)
<img width="1090" height="585" alt="image" src="https://github.com/user-attachments/assets/abd53b17-74c8-4e68-a24a-5b296014f78b" />


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
<img width="665" height="160" alt="image" src="https://github.com/user-attachments/assets/b5a87c20-9199-4795-b016-ea711ac68490" />

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
<img width="565" height="160" alt="image" src="https://github.com/user-attachments/assets/9e75cd29-423f-444a-8422-a0f2742c3a66" />

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
<img width="1084" height="160" alt="image" src="https://github.com/user-attachments/assets/c81fb834-19c0-4558-9e07-24956d57e27d" />

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
<img width="670" height="160" alt="image" src="https://github.com/user-attachments/assets/48868375-e45f-4c57-8480-36e16582db1b" />

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
<img width="640" height="159" alt="image" src="https://github.com/user-attachments/assets/3a9d0bb7-515c-4e9c-b293-dbd15c104a40" />

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
<img width="581" height="210" alt="image" src="https://github.com/user-attachments/assets/7cd7ed98-6882-4f8b-aa3a-63d5e4ba47bd" />
<img width="553" height="166" alt="image" src="https://github.com/user-attachments/assets/f77cc3d4-7a18-4167-baaf-4a0f916fdb31" />

</details>
<img width="612" height="287" alt="image" src="https://github.com/user-attachments/assets/632e698c-f392-434e-b20c-13a72cd96c8e" />
<img width="863" height="356" alt="image" src="https://github.com/user-attachments/assets/5248bd9e-baa2-4a57-a7b9-845f2d1f62d2" />


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
<img width="946" height="160" alt="image" src="https://github.com/user-attachments/assets/a8cdbd9d-2bb0-4eff-94e6-d031aa0e9699" />

</details>
<img width="577" height="339" alt="image" src="https://github.com/user-attachments/assets/10a97d3e-81ad-4711-a42c-4b7acd9be9f2" />

### Orders with no score/review record
- Orders with no score/review record occupy a very small fraction of the data set: 683 records, so they have been excluded from further investigation as they lack volume for meaningful analysis.
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

<img width="1050" height="161" alt="image" src="https://github.com/user-attachments/assets/77e3f5be-91e5-4bbb-8059-5c146cb3b554" />

</details>
 
## 结论
- 发现1
- 发现2
