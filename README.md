# 项目标题

## 背景
一段话描述项目

## 业务表现类
- 销量/GMV趋势（按月）
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
- 销量前十产品、品类
```sql
SELECT 
pcnt.product_category_name_english AS product_catagory,
COUNT(ooid.product_id) AS sales_qty,
ROUND(SUM(ooid.price),2) AS product_gmv
FROM olist_order_items_dataset ooid 
INNER JOIN olist_products_dataset opd
ON ooid.product_id = opd.product_id
INNER JOIN product_category_name_translation pcnt
ON opd.product_category_name  = pcnt.product_category_name 
GROUP BY product_catagory ORDER BY product_gmv DESC LIMIT 10;
```
- 各州销售额分布
- 卖家集中度（头部卖家占比）

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
