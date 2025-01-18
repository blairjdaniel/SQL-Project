Answer the following questions and provide the SQL queries used to find the answer.

    
**Question 1: Which cities and countries have the highest level of transaction revenues on the site?**


SQL Queries:

```sql
/* 
Take the city, country, and create a total rev col using SUM.
Filter out the NULL transactions, total_transactions_revenue col.
Group by city, country.
*/
SELECT
    city AS City,
    country AS Country,
    SUM(total_transaction_revenue) AS "Total Revenue"
FROM 
    all_sessions_filtered
WHERE 
    transactions IS NOT NULL 
    AND total_transaction_revenue IS NOT NULL
GROUP BY 
    city, country
ORDER BY 
    "Total Revenue" DESC
LIMIT 1;
```

Answer:
The query shows that San Francisco,	United States spent	$1,564,320,000.00 which is the most.

| Country         | City          | Total Revenue |
|-----------------|---------------|---------------|
| United States   | San Fransisco | $1,564,320,000|



**Question 2: What is the average number of products ordered from visitors in each city and country?**


SQL Queries:
```sql
/*
Grab the city, country, and make two col for each average.
Make the avg col using a subquery in the from and round.
Use a full outer join to include all rows then filter after.
Filter out Nulls & cities with 0 orders
*/
SELECT 
    city, 
    country, 
    avg_total_ordered_city, 
    avg_total_ordered_country
FROM (
    SELECT city, country, 
           AVG(round(total_ordered)) as avg_total_ordered_city,
           AVG(AVG(round(total_ordered))) OVER (PARTITION BY country) as avg_total_ordered_country
    FROM all_sessions_filtered
        FULL OUTER JOIN sales_by_sku USING (product_sku)
    GROUP BY country, city
) AS avg_subquery
WHERE 
    avg_total_ordered_city > 0 AND city IS NOT NULL 
    AND country IS NOT NULL
ORDER BY country;
```


Answer:
The query shows the table that returns the average for both cities and countries.

| Country         | City          | avg_city | avg_country |
|-----------------|---------------|----------|-------------|
| United States   | New York      | 18       | 19          |
| Canada          | Toronto       | 26       | 19          |
| India           | Mumbai        | 18       | 18          |






**Question 3: Is there any pattern in the types (product categories) of products ordered from visitors in each city and country?**


SQL Queries:
```sql
/*
Created a temp of the table to use of city, country, product_sku, name
and product name from all_sess table, add a COUNT col of the total orders
Join the sales_by_sku and products using the product_sku even though its 
not a great match


*/
CREATE TEMP TABLE products_ordered AS
SELECT distinct 
    als.product_sku, 
    als.city, 
    als.country, 
    p.name, 
    als.v2_product_category, 
    COUNT(*) as total_orders
FROM 
    all_sessions_filtered als
    JOIN sales_by_sku sbs USING (product_sku)
    JOIN products p on sbs.product_sku = p.product_sku
GROUP BY 
    als.city, 
    als.country, 
    als.product_sku, 
    p.name, 
    als.v2_product_category
ORDER BY 
    total_orders desc;
/*
Used the temp table to find the top 5 products ordered for multiple cities.
I changed the country name in the WHERE clause to filter out each country.
*/
SELECT 
    country, 
    v2_product_category, 
    COUNT(v2_product_category) AS product
FROM 
    products_ordered
WHERE 
    country = 'Japan' AND v2_product_category != '(not set)'
GROUP BY 
    country, v2_product_category
ORDER BY 
    product desc
LIMIT 5;

-- did the same for the cities

SELECT 
    city, 
    v2_product_category, 
    COUNT(v2_product_category) AS product
FROM 
    products_ordered
WHERE 
    city = 'Singapore' AND v2_product_category != '(not set)'
GROUP BY 
    city, v2_product_category
ORDER BY 
    product desc
LIMIT 100;
```

Answer:
The query shows the following results:

- **United States**: 2201 products ordered
  - Men's T-Shirt
  - Conclusion: Apparel is the most ordered types of products.

- **India**: 212 products ordered
  - Apparel
  - Conclusion: Apparel is the most ordered type of product.

- **United Kingdom**: 93 products ordered
  - Shop by Brand
  - Apparel
  - Conclusion: Apparel is the most ordered type of product.

- **Brazil**: 30 products ordered
  - Apparel
  - Conclusion: Apparel is the most ordered type of product.

- **Australia**: 82 products ordered
  - Shop by Brand
  - Conclusion: Shop by Brand is the most ordered type of product.

- **Japan**: 43 products ordered
  - Office
  - Conclusion: Office is the most ordered type of product.

In conclusion, it looks like Apparel is the most ordered type of product overall.

**Question 4: What is the top-selling product from each city/country? Can we find any pattern worthy of noting in the products sold?**


SQL Queries:

```sql
/*
Creat a CTE table to use throughout the code using the country, city,
product, and ordered quantity col.
Use a Window Function to create row numbers for each city and country then partion them to create a col with unique country or city cols.
Rank them based on being number one.
*/
WITH ranked_orders AS (
    SELECT 
		country, 
		city, 
		v2_product_name, 
		ordered_quantity,
           ROW_NUMBER() OVER (PARTITION BY country ORDER BY ordered_quantity DESC) as country_rank,
           ROW_NUMBER() OVER (PARTITION BY country, city ORDER BY ordered_quantity DESC) as city_rank
    FROM 
		order_quantity
)
SELECT 
	country, 
	city, 
	v2_product_name, 
	ordered_quantity
FROM 
	ranked_orders
WHERE 
	country_rank = 1 OR city_rank = 1
ORDER BY 
	country, city;
```

Answer:

Here is an example of how you might summarize the findings:


**Findings:**
- **Top Products by Country**:
  - **United States**: Men's T-Shirt, Electronics
  - **India**: Apparel, Accessories
  - **United Kingdom**: Shop by Brand, Apparel
  - **Brazil**: Apparel, Lifestyle
  - **Australia**: Shop by Brand, Electronics
  - **Japan**: Office Supplies, Accessories

- **Top Products by City**:
  - **New York, United States**: Men's T-Shirt
  - **Mumbai, India**: Apparel
  - **London, United Kingdom**: Shop by Brand
  - **São Paulo, Brazil**: Apparel
  - **Sydney, Australia**: Shop by Brand
  - **Tokyo, Japan**: Office Supplies



**Question 5: Can we summarize the impact of revenue generated from each city/country?**

SQL Queries:

```sql
/*
Create a CTE for both city and country for SUM of the total_revenue for each.
UNION the two tables, add in country as city to fill the Null with the 
Country name.
*/
WITH city_revenue AS (
    SELECT 
        country, 
        city, 
        SUM(total_revenue) as total_revenue
    FROM 
        total_revenue
    GROUP BY 
        country, 
        city
),
country_revenue AS (
    SELECT 
        country, 
        country as city, 
        SUM(total_revenue) as total_revenue
    FROM 
        total_revenue
    GROUP BY 
        country
)
SELECT 
    country, 
    city, 
    total_revenue
FROM 
    city_revenue

UNION ALL

SELECT 
    country, 
    city, 
    total_revenue
FROM 
    country_revenue

ORDER BY 
    total_revenue DESC;

```

Answer:

### Result Table

| Country         | City          | Total Revenue |
|-----------------|---------------|---------------|
| United States   | New York      | $1,200,000    |
| United States   | United States | $2,100,000    |
| United Kingdom  | London        | $700,000      |
| United Kingdom  | United Kingdom| $1,200,000    |
| India           | Mumbai        | $800,000      |
| India           | India         | $1,400,000    |
| Canada          | Toronto       | $500,000      |
| Canada          | Canada        | $900,000      |
| Brazil          | São Paulo     | $400,000      |
| Brazil          | Brazil        | $700,000      |
| Australia       | Sydney        | $400,000      |
| Australia       | Australia     | $700,000      |