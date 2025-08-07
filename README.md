# SQL---Explore-Ecommerce-Dataset
I. Introduction


This analytical framework offers a comprehensive, query‑based approach to understand your eCommerce site's traffic trends, conversion efficiency, channel performance, user behavior segmentation, and product interactions and funnels during early 2017 and summer 2017 periods. It empowers stakeholders to identify actionable insights for marketing optimization, user retention, and conversion rate improvement, using SQL-driven exploratory analysis in BigQuery-based datasets.

II. Requirements


III. Dataset access



IV. Exploring the Dataset

In this project, I will write 08 query in Bigquery base on Google Analytics dataset


-- q1--

SELECT 
         format_date('%Y%m', PARSE_DATE('%Y%m%d', date)) as month
        ,Count(visitNumber) visits
        ,count(totals.pageviews) pageviews
        ,count(totals.transactions) transactions
 FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`
 where _table_suffix between '0101'and '0331'
 group by month 
 order by month;

 Output 

--q2--

with raw_data as (
   SELECT  
      trafficSource.source as source
      ,count(visitNumber) total_visit
      ,count(totals.bounces) num_bounce
   FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`
   group by trafficSource.source
 )
 select
      source
      ,total_visit
      ,num_bounce
      ,round (SAFE_DIVIDE(num_bounce,total_visit)*100,2) bounce_rate
   from raw_data 
 order by total_visit desc;

 -- q3--

with month_revenue as(

   SELECT 
      'Month' as time_type -- Month_revenue
      ,format_date('%Y%m', PARSE_DATE('%Y%m%d', date)) time_table
      ,trafficSource.source as key_source
      ,round(sum(product.productRevenue)/1000000,2) as revenue
   FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`,
      UNNEST (hits) hits,
      UNNEST (hits.product) product
      Group by time_type, time_table,key_source
   )
, week_revenue as ( -- Week_revenue
      select 
         'Week' as time_type
         ,format_date('%Y%W',PARSE_DATE('%Y%m%d', date)) as time_table
         ,trafficSource.source as key_source
         ,round(sum(product.productRevenue)/1000000,2) as revenue
      FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`,
      UNNEST (hits) hits,
      UNNEST (hits.product) product 
      Where product.productRevenue is not null 
      Group by time_type, time_table, key_source
)


select
      time_type
      ,time_table
      , key_source 
      , revenue
from month_revenue
UNION All
select 
      time_type
      , time_table
      , key_source 
      , revenue
from week_revenue
order by time_type desc
       ,time_table, revenue desc;


---q4---
with purchase_data as ( -- STEP 1: TINH AVG_PAGEVIEW CUA PURCHASE DATA TYPE
  SELECT 
     format_date('%Y%m',PARSE_DATE('%Y%m%d', date)) as month
        ,sum(totals.pageviews)/count (fullVisitorId) as avg_pageviews_purchaser
      FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`,
       UNNEST (hits) hits,
    UNNEST (hits.product) product
        where _table_suffix between '0601'and '0731'
              and totals.transactions >=1
              and product.productRevenue is not null
              Group by month
  )
  ,non_purchase_data  as ( -- STEP 1: TINH AVG_PAGEVIEW CUA NON-PURCHASE DATA TYPE
    select
      format_date('%Y%m',PARSE_DATE('%Y%m%d', date)) as month
      ,sum(totals.pageviews)/count (fullVisitorId) as avg_pageviews_non_purchaser
     FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`,
       UNNEST (hits) hits,
    UNNEST (hits.product) product
        where _table_suffix between '0601'and '0731'
            and totals.transactions is null
            and product.productRevenue is null
            Group by month
)

Select 
   COALESCE (p.month, np.month) as month
   ,p.avg_pageviews_purchaser as avg_pageviews_purchaser
   ,np.avg_pageviews_non_purchaser as avg_pageviews_non_purchaser
from purchase_data p
FULL OUTER JOIN non_purchase_data np --- outer join cho 2 bang
   using(month)
order by month;
    

--- q5---

SELECT 
     format_date('%Y%m',PARSE_DATE('%Y%m%d', date)) as month
    ,sum(totals.transactions)/count(fullVisitorId) as avg_total_transactions_per_user
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`,
    UNNEST (hits) hits,
    UNNEST (hits.product) product
  where _table_suffix between '01'and '31'
        and totals.transactions >=1
        and product.productRevenue is not null
group by month, fullVisitorId
order by month;

--- q6---

SELECT
 format_date('%Y%m',PARSE_DATE('%Y%m%d', date)) as month
    ,sum(product.productRevenue)/SUM(totals.visits)/1000000 as avg_spend_per_session
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`,
    UNNEST (hits) hits,
    UNNEST (hits.product) product
  where _table_suffix between '01'and '31'
        and product.productRevenue is not null
        and product.productRevenue is not null
group by month
order by month;

--- q7---
with buyers as (
SELECT 
     format_date('%Y%m',PARSE_DATE('%Y%m%d', date)) as month
    ,fullVisitorId
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`,
    UNNEST (hits) hits,
    UNNEST (hits.product) product
  where _table_suffix between '01'and '31'
        and product.v2ProductName="YouTube Men's Vintage Henley"
        and totals.transactions >=1 
        and product.ProductRevenue is not null
),
other as (
SELECT
      distinct fullVisitorId
      ,product.v2ProductName as other_purchased_products
      ,sum(product.productQuantity) product_quantity
   FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`,
    UNNEST (hits) hits,
    UNNEST (hits.product) product
    where _table_suffix between '01'and '31'
    and product.v2ProductName !="YouTube Men's Vintage Henley"
    and totals.transactions >=1 
        and product.ProductRevenue is not null
   group by fullVisitorId,product.v2ProductName
)
SELECT 
      o.other_purchased_products
      ,o.product_quantity
from other  o
JOIN buyers b
      ON o.fullVisitorId =b.fullVisitorId
GROUP BY o.other_purchased_products,o.product_quantity
ORDER BY o.product_quantity desc;

