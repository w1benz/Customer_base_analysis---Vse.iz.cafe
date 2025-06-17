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
