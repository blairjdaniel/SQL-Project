What issues will you address by cleaning the data?

**Loading Data**
    - I noticed right away the col names were a mix of snake & camel so
    I formatted them all to snake but I did it manually in PgAdmin
    - I then looked at what was in the col as far as data types then
    changed them accordingly
    
**Question ONE**
    1. I wanted to look at the country col and standardize any col that might use shorform like U.S or U.K.
    2. I found some col had ('not set') so I changed it
    3. I created a view table that filtered out the Null or 'not set'<br><br>
**Question TWO**
    1. I wanted to see which product col from which table to use.
    2. show how many rows in each col.<br><br>
**Question THREE**<br><br>
**Question FOUR**
    1. Start by checking for Nulls or values that seem off -- no nulls --   <  1 has 512 rows, I can take these out with a view table
    2. Create a view table for ordered_quantity > 0
    3. Need to get the product_price col to normal values<br><br>
**Question FIVE**
    1. Create a clean analytics View with non null revenues. 
    2. Look for how many cols have a country match to revenue using 
    the analytics table<br><br>
**New Question 1**
    1.Check for the primary key of the analytics_2 table to join with all_sessions_filtered table.
    I'm going to assume the visit_id is as close to a primary key as possible and make a cte later filtering out all NULL visit_id or have duplicates
    2.
    

Queries:
Below, provide the SQL queries you used to clean your data.<br>
**Question ONE**<br>
1.
```sql
select * 
from all_sessions
select country = 
	CASE
    	WHEN country IN ('USA', 'United States') THEN 'United States'
    	WHEN country = 'UK' THEN 'United Kingdom'
    	ELSE country
    END
from all_sessions 
```
2. 
```sql
select *
from all_sessions
where country = '(not set)' 

select *
from all_sessions
where city = '(not set)' 
```
3. 
```sql
create or replace view all_sessions_filtered as
select *
from all_sessions
where country != '(not set)'
	and city not in ('(not set)', 'not available in demo dataset')
```

**Question TWO**<br>
1.
```sql
SELECT 
    asf.product_sku AS all_sessions_product_sku, 
    p.product_sku AS products_product_sku, 
    sbs.product_sku AS sales_by_sku_product_sku, 
    p.name, 
    asf.v2_product_name
FROM products p
    RIGHT JOIN all_sessions_filtered asf ON p.product_sku = asf.product_sku
	LEFT JOIN sales_by_sku sbs ON asf.product_sku = sbs.product_sku
ORDER BY all_sessions_product_sku;
```
2.
```sql
-- 1092 
SELECT COUNT(product_sku)
FROM products
--6478
SELECT COUNT(product_sku)
FROM all_sessions_filtered
--462
SELECT COUNT(product_sku)
FROM sales_by_sku
```
**Question FOUR**<br>
1.
```sql
SELECT 
    asf.product_sku, 
    asf.v2_product_name, 
    p.ordered_quantity
FROM products p
	JOIN all_sessions_filtered asf ON p.product_sku = asf.product_sku
WHERE 
    ordered_quantity IS NULL

SELECT 
    asf.product_sku, 
    asf.v2_product_name, 
    p.ordered_quantity
FROM products p
	JOIN all_sessions_filtered asf ON p.product_sku = asf.product_sku
WHERE 
    ordered_quantity < 1
```
2.
```sql
CREATE OR REPLACE VIEW order_quantity AS
SELECT 
    asf.product_sku, 
    asf.v2_product_name, 
    p.ordered_quantity, 
    asf.country, 
    asf.city, 
    asf.v2_product_category 
FROM 
    products p
	JOIN all_sessions_filtered asf ON p.product_sku = asf.product_sku
WHERE 
    ordered_quantity > 1
ORDER BY 
    ordered_quantity DESC;
```
3. 
```sql
UPDATE 
    all_sessions_filtered
SET 
product_price = product_price / 10;
```

**Question FIVE**<br>
1.
```sql
sql
CREATE OR REPLACE VIEW analytics AS
SELECT 
	COUNT(revenue)
FROM 
	analytics_2
WHERE 
    revenue IS NOT NULL
```
2. 
```sql
SELECT 
	country, 
	city, 
	SUM(a.revenue) as total_revenue
FROM 
	analytics a
	FULL OUTER JOIN all_sessions_filtered asf ON a.full_visitor_id = asf.full_visitor_id
WHERE 
    revenue IS NOT NULL
GROUP BY 
    country, city
ORDER BY 
    total_revenue DESC;
```

**New Question 1**
```sql

SELECT
	COUNT(distinct full_visitor_id)
FROM 
	analytics_2 -- 120018 unique
SELECT 
	COUNT(distinct visit_id)
FROM 
	analytics_2 -- 148642
```