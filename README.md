# Customer_base_analysis---Vse.iz.cafe

### Dashboard Link: https://datalens.yandex/8763xcv9j7msu

This project involved analyzing key business metrics using PostgreSQL queries and visualizing the results in Yandex DataLens. The goal was to track performance trends and identify insights for decision-making.

### Tasks Completed:

**Wrote SQL queries to calculate core metrics:**
- DAU (Daily Active Users)
- Conversion Rate
- Average Purchase Value
- LTV (Lifetime Value)
- Retention Rate

Built a DataLens dashboard with interactive visualizations (line charts, tables) to monitor trends.
Performed time-based analysis (May–June) to detect anomalies and growth patterns.

**Key Insights:**
- DAU trends: Increasing or declining active users.
- Conversion anomalies: Spikes or drops in user conversion.
- Average purchase trends: Changes in spending behavior.
- Retention dynamics: How well users stick over time.
- The dashboard provides a clear, actionable overview of business performance.

---

## SQL Queries Used in the Project
**Below are the key PostgreSQL queries I wrote to calculate the business metrics:**

### DAU - Number of unique daily active users 
```
SELECT log_date,
    COUNT(DISTINCT user_id) AS DAU
from rest_analytics.cities AS c JOIN rest_analytics.analytics_events AS a
on c.city_id = a.city_id
WHERE log_date  BETWEEN '2021-05-01' AND '2021-06-30' AND c.city_id = 6 AND event = 'order'
GROUP BY log_date
ORDER BY log_date
```
---
### CR by days
```
SELECT log_date,
    ROUND(COUNT(DISTINCT user_id) FILTER ( WHERE event = 'order' ) / COUNT(DISTINCT user_id)::numeric, 2) AS CR 
FROM rest_analytics.analytics_events
WHERE log_date BETWEEN '2021-05-01' AND '2021-06-30' AND city_id = 6 
GROUP BY log_date
ORDER BY log_date
```
---
### TOP 3 restaurants by LTV
```
-- We calculate the commission amount for each order, select orders by date and city
WITH orders AS
    (SELECT analytics_events.rest_id,
            analytics_events.city_id,
            revenue * commission AS commission_revenue
     FROM rest_analytics.analytics_events
     JOIN rest_analytics.cities ON analytics_events.city_id = cities.city_id
     WHERE revenue IS NOT NULL
         AND log_date BETWEEN '2021-05-01' AND '2021-06-30'
         AND city_name = 'Саранск')
SELECT o.rest_id,
    chain AS "Network name",
    type AS "Kitchen type",
    ROUND(SUM(commission_revenue)::numeric, 2) AS LTV
FROM orders AS o JOIN rest_analytics.partners AS p
ON o.rest_id = p.rest_id  AND o.city_id = p.city_id
GROUP BY 1, 2, 3 
ORDER BY LTV DESC
LIMIT 3
```
---
### LTV calculation for restaurants — most popular dishes
```
-- We calculate the commission amount for each order, select orders by date and city
WITH orders AS
    (SELECT a.rest_id,
            a.city_id,
            a.object_id,
            revenue * commission AS commission_revenue
     FROM rest_analytics.analytics_events AS a
     JOIN rest_analytics.cities AS c ON a.city_id = c.city_id
     WHERE revenue IS NOT NULL
         AND log_date BETWEEN '2021-05-01' AND '2021-06-30'
         AND city_name = 'Саранск'), 
-- We calculate the two restaurants with the highest LTV 
top_ltv_restaurants AS
    (SELECT o.rest_id,
            chain,
            type,
            ROUND(SUM(commission_revenue)::numeric, 2) AS LTV
     FROM orders AS o JOIN rest_analytics.partners AS p 
     ON o.rest_id = p.rest_id AND o.city_id = p.city_id
     GROUP BY 1, 2, 3
     ORDER BY LTV DESC
     LIMIT 2)
SELECT t.chain AS "Chain name",
    d.name AS "Dish name",
    CASE WHEN d.spicy = 1 THEN 1 ELSE 0 END AS spicy,
    CASE WHEN d.fish = 1 THEN 1 ELSE 0 END AS fish,
    CASE WHEN d.meat = 1 THEN 1 ELSE 0 END AS meat,
    ROUND(SUM(o.commission_revenue)::numeric, 2) AS LTV
FROM top_ltv_restaurants AS t
JOIN orders AS o ON t.rest_id = o.rest_id
JOIN rest_analytics.dishes AS d ON o.rest_id = d.rest_id AND o.object_id = d.object_id
GROUP BY 1, 2, 3, 4, 5
ORDER By LTV DESC
```
---
### Average check (May - June)
```
-- We calculate the commission amount for each order, select orders by date and city
WITH orders AS
    (SELECT *,
            revenue * commission AS commission_revenue
     FROM rest_analytics.analytics_events
     JOIN rest_analytics.cities ON analytics_events.city_id = cities.city_id
     WHERE revenue IS NOT NULL
         AND log_date BETWEEN '2021-05-01' AND '2021-06-30'
         AND city_name = 'Саранск')
SELECT 
    DATE_TRUNC('month', log_date)::date AS "Month",
    COUNT(DISTINCT order_id) AS "Order count",
    ROUND(SUM(commission_revenue)::numeric, 2) AS "Commission amount",
    ROUND(SUM(commission_revenue)::numeric / COUNT(DISTINCT order_id), 2) AS "Average check"
FROM orders
GROUP BY 1
ORDER BY 1
```
---
