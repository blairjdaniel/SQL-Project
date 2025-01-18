What are your risk areas? Identify and describe them.
1. Eliminate nulls and non-country, non-city names in all_sessions
then create a view with cleaned data
2. Datatype for the Date col is integer, so I am going to change it 
datetime

QA Process:
Describe your QA process and include the SQL queries used to execute it.
1.
```sql
-- make the all_sessions_filtered table
SELECT
	* 
FROM
	all_sessions
-- we see (not set) and 'not available in demo dataset', lets filter those out
-- 24 rows with not set and 354 rows in the city column
select *
from all_sessions
where country = '(not set)' 

select *
from all_sessions
where city = '(not set)' 

-- Create a view table with the non set and not available in demo dataset rows taken out

create or replace view all_sessions_filtered as
select 
	*
from 
	all_sessions
where 
	country != '(not set)'

-- Standardized the naming of the countries
select 
	* 
from 
	all_sessions_filtered
select 
	country = 
	CASE
    	WHEN country IN ('USA', 'United States') THEN 'United States'
    	WHEN country = 'UK' THEN 'United Kingdom'
    	ELSE country
    END
from 
	all_sessions_filtered 

-- check view for not set and not available in demo dataset. All empty!
SELECT
	*
FROM 
	all_sessions_filtered
WHERE 
	country = '(not set)'
	--city = '(not set)'
	--city = 'not available in demo dataset'    
```
2.
```sql
/* Look at the date col in analytics to see if anything is off
date col is data type integer, lets change it datetime
*/
-- check the schema for datatype, find that its integer
SELECT *
FROM information_schema.columns
WHERE table_name = 'analytics_2';

-- create a cte to cast and change the date col to datetime
WITH date_changed AS (
    SELECT 
        TO_DATE(CAST(date AS TEXT), 'YYYYMMDD') AS closed_date
    FROM 
        analytics_2
    WHERE 
        date IS NOT NULL
        AND revenue IS NOT NULL --take out the null days of zero revenue
)
-- use the CTE to find the count of how many 
WITH date_changed AS (
    SELECT 
        TO_DATE(CAST(date AS TEXT), 'YYYYMMDD') AS closed_date
    FROM 
        analytics_2
    WHERE 
        date IS NOT NULL
		AND revenue IS NOT NULL --take out the null days of zero revenue
)
SELECT DISTINCT *
FROM (
    SELECT 
        closed_date, 
        COUNT(*) AS row_count
    FROM 
        date_changed
    GROUP BY 
        closed_date
    ORDER BY 
        closed_date DESC
) subquery
WHERE 
    closed_date > '2017-01-01'

-- check for abnormalitites using aggregate functions
WITH date_changed AS (
    SELECT 
        TO_DATE(CAST(date AS TEXT), 'YYYYMMDD') AS closed_date
    FROM 
        analytics_2
    WHERE 
        date IS NOT NULL
        AND revenue IS NOT NULL --take out the null days of zero revenue
),
subquery AS (
    SELECT 
        closed_date, 
        COUNT(*) AS row_count
    FROM 
        date_changed
    GROUP BY 
        closed_date
    ORDER BY 
        closed_date DESC
)
SELECT 
    ROUND(AVG(row_count), 2) AS row_count,
    CASE
        WHEN row_count > 165.11 THEN 'Above average row count.'
        WHEN row_count < 165.11 THEN 'Below average row count.'
        ELSE 'Average row count.'
    END AS avg_comparison,
	closed_date
FROM 
    subquery
WHERE 
    closed_date > '2017-01-01'
GROUP BY subquery.row_count, avg_comparison, closed_date 

-- show as percent change
WITH date_changed AS (
    SELECT 
        TO_DATE(CAST(date AS TEXT), 'YYYYMMDD') AS closed_date
    FROM 
        analytics_2
    WHERE 
        date IS NOT NULL
        AND revenue IS NOT NULL --take out the null days of zero revenue
),
subquery AS (
    SELECT 
        closed_date, 
        COUNT(*) AS row_count
    FROM 
        date_changed
    GROUP BY 
        closed_date
    ORDER BY 
        closed_date DESC
)
SELECT 
    ROUND(AVG(row_count) OVER (), 2) AS row_count_avg,
    CASE
        WHEN row_count > 165.11 THEN 'Above average row count.'
        WHEN row_count < 165.11 THEN 'Below average row count.'
        ELSE 'Average row count.'
    END AS avg_comparison,
    closed_date,
    ROUND(100 * (row_count::numeric / LAG(row_count) OVER (ORDER BY closed_date) - 1), 2) || '%' AS daily_percent_change
FROM 
    subquery
WHERE 
    closed_date > '2017-01-01'
ORDER BY 
    closed_date DESC;

/*Found these strange occurences in the analytics table
"2017-05-08"	"173.97%" 
"2017-07-05"	"241.79%"
"2017-05-30"	"166.28%"
"2017-06-11"	"121.05%"
"2017-07-10"	"102.08%"
"2017-07-31"	"130.94%"*/

--Look at what was happening on one of the busy days
SELECT
	Distinct asf.visit_id, *
FROM 
	analytics_2 a
JOIN 
	all_sessions_filtered asf USING(visit_id)
WHERE 
	a.date = '20170705'
```