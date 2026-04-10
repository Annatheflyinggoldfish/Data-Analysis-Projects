# 项目标题

## 背景
一段话描述项目

## 业务表现类
- Monthly GMV
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
- TOP 10 best-selling product categories and GMV
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
- GMV by state
```sql
WITH payment AS
(SELECT order_id,SUM(oopd.payment_value) AS order_payment
FROM olist_order_payments_dataset oopd GROUP BY oopd.order_id
)
SELECT 
ocd.customer_state AS state,
COUNT(*) AS Qty,
ROUND(SUM(p.order_payment),2) AS state_gmv
FROM olist_customers_dataset ocd
JOIN olist_orders_dataset ood
ON ocd.customer_id = ood.customer_id
JOIN payment p
ON ood.order_id = p.order_id
GROUP BY state
ORDER BY state_gmv DESC;
```

- TOP 10 seller
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


- Top 10 Seller GVM
```sql
SELECT ROUND(SUM(gmv),2) AS seller_gmv FROM top10_sellers;
```
<img width="210" height="60" alt="image" src="https://github.com/user-attachments/assets/0d20af53-4fd2-4dbe-838f-40938c4fb274" />



- Total GVM: 16008872.12
```sql
SELECT ROUND(SUM(order_payment),2) AS total_gmv FROM payment;
```
<img width="206" height="61" alt="image" src="https://github.com/user-attachments/assets/bbc050d2-7ac7-4081-8fb0-6d7ea1cd4238" />



- Top 10 Sellers' Contribution to Total GMV
```sql
SELECT CONCAT(ROUND(SUM(gmv)/(SELECT SUM(order_payment) FROM payment),2),'%') AS CR10 FROM top10_sellers;
```
<img width="174" height="61" alt="image" src="https://github.com/user-attachments/assets/e88c7ecc-893b-4ced-80bf-b009fefa51fe" />



## 客户行为类
- Repeat Purchase Rate
```sql
WITH T AS
(SELECT customer_unique_id,COUNT(customer_unique_id) AS order_count
FROM olist_customers_dataset ocd
GROUP BY customer_unique_id 
ORDER BY order_count DESC)
SELECT CONCAT(ROUND(COUNT(*)/(SELECT COUNT(DISTINCT customer_unique_id) FROM olist_customers_dataset ocd)*100,2),'%') AS repeat_purchase_rate 
FROM T WHERE order_count >= 2;
```
<img width="280" height="60" alt="image" src="https://github.com/user-attachments/assets/326c3591-58c1-4b12-9f11-c57bdfe8bc60" />
 
- Average Inter-purchase Time
- Average Order Value Distribution 客单价分布
- Regional Analysis 用户地域分布及消费习惯差异
- Time Latency for Customer Feedback 购买到评论的时间间隔

## 物流类
- 实际送达 vs 预估送达的准时率
- 各州物流时效差异
- 物流延误和差评的相关性

## 评论/满意度类
- 评分分布（Olist的评分呈现两极化还是集中化）
- 评分和物流延误的关系
- 评论回复率及回复时间对评分的影响

## 交叉分析类
- 高价产品是否评分更高
- 节假日前后购买行为和评分变化
- 卖家评分和销量的关系

## 工具
MySQL, Tableau Public

## 结论
- 发现1
- 发现2
