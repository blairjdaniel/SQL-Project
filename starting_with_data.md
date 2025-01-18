**Question 1:** Which country and city spent the most time per viewing online?

**SQL Queries:**

```sql
/*
Look into page views and time_on_site, check for which country spends the most time on site. Build the CTE first to make the analytics col easy to 
work with.
There are only 1264 rows with no null values
*/
WITH analytics_primary AS (
    SELECT 
        DISTINCT visit_id, 
        page_views, 
        CAST(time_on_site / 60 AS INTEGER) AS time_on_site_minutes
    FROM
        analytics_2
    WHERE 
        visit_id IS NOT NULL
        AND page_views IS NOT NULL
        AND time_on_site IS NOT NULL
)
SELECT 
    ap.visit_id,
    ap.page_views,
    ap.time_on_site_minutes,
    asf.country,
    asf.city,
    CAST(SUM(ap.time_on_site_minutes / ap.page_views) AS INTEGER) AS average_minutes_per_view
FROM 
    analytics_primary ap
    JOIN all_sessions_filtered asf ON ap.visit_id = asf.visit_id
GROUP BY 
    ap.visit_id,
    ap.page_views,
    ap.time_on_site_minutes,
    asf.country,
    asf.city
ORDER BY average_minutes_per_view DESC
LIMIT 5;
```
**Answer:** Madrid, Spain spends the most time online per page view.

|   Country    |    City        | AVG Time Online   |
|--------------|----------------|-------------------|
| Spain        | Madrid         | 14                |
| Peru         | La Victoria    | 12                |
| United States| Dallas         | 11                |
| Japan        | Sakai          | 9                 |
| India        | Mumbai         | 9                 |


**Question 2:** What did the customers from the above answer buy?

**SQL Queries:**
```sql
/*
add the product name col from the all_sessions_filtered table
*/
WITH analytics_primary AS (
    SELECT 
        distinct visit_id, 
        page_views, 
        CAST(time_on_site / 60 AS INTEGER) AS time_on_site_minutes
    FROM
        analytics_2
    WHERE 
        visit_id IS NOT NULL
        AND page_views IS NOT NULL
        AND time_on_site IS NOT NULL
)
SELECT 
    ap.visit_id,
    ap.page_views,
    ap.time_on_site_minutes,
    asf.country,
    asf.city,
	asf.v2_product_name,
    CAST(SUM(ap.time_on_site_minutes / ap.page_views) AS INTEGER) AS average_minutes_per_view
FROM 
    analytics_primary ap
    JOIN all_sessions_filtered asf ON ap.visit_id = asf.visit_id
GROUP BY 
    ap.visit_id,
    ap.page_views,
    ap.time_on_site_minutes,
    asf.country,
    asf.city,
	asf.v2_product_name
ORDER BY average_minutes_per_view DESC
LIMIT 15;
```

**Answer:**
|   Country    |    City        | Purchase           |
|--------------|----------------|--------------------|
| Spain        | Madrid         |Women's Zip Jacket  |
| Peru         | La Victoria    | NO PURCHASE        |
| United States| Dallas         |Pen & Notebook      |
| Japan        | Sakai          |Leatherette NoteBook|
| India        | Mumbai         |Google NoteBook     |




**Question 3:** How did they find the website?

**SQL Queries:**
```sql
/*
add the channel_grouping col plus count it to find the totals
*/
WITH analytics_primary AS (
    SELECT 
        distinct visit_id, 
        page_views
    FROM
        analytics_2
    WHERE 
        visit_id IS NOT NULL
        AND page_views IS NOT NULL
        AND time_on_site IS NOT NULL
)
SELECT 
	asf.city,
    asf.country,
    asf.channel_grouping,
	COUNT(channel_grouping) AS channel_grouping
   
FROM 
    analytics_primary ap
    JOIN all_sessions_filtered asf ON ap.visit_id = asf.visit_id
GROUP BY 
    asf.country,
    asf.city,
    asf.channel_grouping
HAVING city IN ('Madrid', 'La Victoria', 'Dallas', 'Sakai', 'Mumbai')
ORDER BY COUNT(channel_grouping) DESC
LIMIT 15;

```
**Answer:** This table shows the way each viewer found the website by country and city.

|City|Country|Channel|Total|
|----|-------|-------|-----|
|Madrid|Spain|Direct |2|
|Madrid|Spain|Organic Search|2|
|La Victoria|Peru|Organic Search|5|
|La Victoria|Peru|Affiliates|1|
|Dallas|United States|Organic Search|7|
|Dallas|United States|Direct|2|
|Sakai|Japan|Direct|1|
|Mumbai|India|Organic Search|7|
|Mumbai|India|Affiliates|1|
