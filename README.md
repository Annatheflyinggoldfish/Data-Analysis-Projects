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
seller                          |seller_gmv|
--------------------------------+----------+
7c67e1448b00f6e969d365cea6b010ab| 507166.91|
1025f0e2d44d7041d6cf58b6550e0bfa| 308222.04|
4a3ca9315b744ce9f8e9374361493884| 301245.27|
1f50f920176fa81dab994f9023523100| 290253.42|
53243585a1d6dc2643021fd1853d8905| 284903.08|
da8622b14eb17ae2831f4ac5b9dab84a| 272219.32|
4869f7a5dfa277a7dca6462dcf3b52b2| 264166.12|
955fee9216a65b617aa5c0531780ce60|  236322.3|
fa1c13f2614d7b5c4749cbc52fecda94| 206513.23|
7e93a43ef30c4f03f38b393420bc753a| 185134.21|

Top 10 Seller GVM
```sql
SELECT ROUND(SUM(gmv),2) AS seller_gmv FROM top10_sellers；
```
<img width="210" height="61" alt="image" src="https://github.com/user-attachments/assets/56d5d620-c482-4ad1-a32c-4582fe83e92b" />


Total GVM: 16008872.12
```sql
SELECT ROUND(SUM(order_payment),2) AS total_gmv FROM payment;
```
<img width="205" height="55" alt="image" src="https://github.com/user-attachments/assets/f882e8ed-d549-42c2-8bf5-5397b12cd0c9" />


Top 10 Sellers' Contribution to Total GMV
```sql
SELECT CONCAT(ROUND(SUM(seller_gmv)/(SELECT SUM(order_payment) FROM payment),2),'%') FROM top10_sellers;
```
CR10 |
-----+
0.18%|


## 客户行为类
- 复购率（Olist复购率极低，这本身就是一个有趣的发现）
- 客单价分布
- 用户地域分布及消费习惯差异
- 购买到评论的时间间隔

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
