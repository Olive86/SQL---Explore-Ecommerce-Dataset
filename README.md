# SQL---Explore-Ecommerce-Dataset
I. Introduction


This analytical framework offers a comprehensive, query‑based approach to understand your eCommerce site's traffic trends, conversion efficiency, channel performance, user behavior segmentation, and product interactions and funnels during early 2017 and summer 2017 periods. It empowers stakeholders to identify actionable insights for marketing optimization, user retention, and conversion rate improvement, using SQL-driven exploratory analysis in BigQuery-based datasets.

II. Requirements


III. Dataset access



IV. Exploring the Dataset

In this project, I will write 08 query in Bigquery base on Google Analytics dataset

Query 01: calculate total visit, pageview, transaction for Jan, Feb and March 2017 (order by month)

<img width="609" height="203" alt="image" src="https://github.com/user-attachments/assets/7e318565-ae94-4f14-9180-df6c57829890" />

Output

<img width="733" height="161" alt="image" src="https://github.com/user-attachments/assets/c9933e30-6d91-426b-9c2f-c6b2208c0b9f" />


Query 02: Bounce rate per traffic source in July 2017 (Bounce_rate = num_bounce/total_visit) (order by total_visit DESC)


<img width="611" height="271" alt="image" src="https://github.com/user-attachments/assets/ef5482d3-d0ae-4472-8738-c46af5f8b08f" />



Output

<img width="578" height="388" alt="image" src="https://github.com/user-attachments/assets/15380c16-1f16-4de5-acd9-b6a48041c877" />


Query 3: Revenue by traffic source by week, by month in June 2017


Output

