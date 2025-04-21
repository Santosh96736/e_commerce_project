# 🛒 E-Commerce Sales Analysis (Target Brazil)
Analyzing 100K+ orders from a leading Brazilian marketplace using SQL & Python

## TABLE CONTENTS
1. [PROJECT OVERVIEW](#1-Project-overview)
2. [TECH STACK](#2-Tech-Stack)
3. [PROJECT OBJECTIVE](#3-Project-Objective)
4. [DATABASE SCHEMA](#4-Database-Schema)
5. [SQL QUERIES](#5-SQL-Queries)
6. [JUPYTER CODE](#6-Jupter-Code)
7. [KEY FINDINGS](#7-Key-Findings)
8. [SUGGESTION AND RECOMMENDATION](#8-Suggestion-and-Recommendation)
9. [CONTACT](#9-Contact)


## 1. Project Overview
-   This project is based on the Brazilian E-Commerce Public Dataset by Olist, which includes over 100,000 orders from 2016 to 2018, made across multiple marketplaces in 
    Brazil. The dataset was made publicly available by Olist, a Brazilian e-commerce platform that connects small businesses to major online retail channels.

    Using MySQL, and Python, this project aims to explore and analyze various aspects of the e-commerce business, including customer behavior, order trends, product 
    performance, and seller contributions.

    The project focuses on turning raw transactional data into actionable business insights through structured SQL queries and Python.

-   To download the dataset click here : [e-Commerce (Target) Sales Dataset - Kaggle](https://www.kaggle.com/datasets/ujjwalinsights/target-case-study-using-sql)

## 2. Tech Stack

- **Languages:** SQL, Python
- **Database:** MySQL
- **Python Libraries:** Pandas, Matplotlib, Seaborn, SQLAlchemy
- **Tools:** Jupyter Notebook, MySQL Workbench

## 3. Project Objectives

- Understand order volume, revenue, and trends over time.
- Identify top-performing product categories and sellers.
- Analyze delivery performance and delays.
- Evaluate payment methods and financial metrics.
- Detect patterns in customer purchases and retention.

## 4. Database Schema
The dataset includes the following tables:

- `customers`
- `orders`
- `order_items`
- `order_payments`
- `order_reviews`
- `products`
- `sellers`
- `product_category_name_translation`
- `geolocation`
  
A comprehensive schema diagram was created in MySQL Workbench, with primary and foreign keys declared across all tables.

## 5. SQL Queries

### CREATE DATABASE

*    ```sql
     CREATE DATABASE ecommerce;
     ```
     
### USE DATABASE

*   ```sql
    USE ecommerce;
    ```
### CREATE TABLES
*   ```sql
    CREATE TABLE geolocation(
    geolocation_zip_code_prefix INT,
    geolocation_lat DECIMAL(18,15),
    geolocation_lng DECIMAL(18,15),
    geolocation_city VARCHAR(40),
    geolocation_state VARCHAR(5),
    PRIMARY KEY (geolocation_zip_code_prefix, geolocation_lat, geolocation_lng)
    );
    ```

### EASY QUERIES

*   ```sql
    -- LIST ALL UNIQUE CITIES WHERE CUSTOMERS ARE LOCATED

    SELECT DISTINCT customer_city
    FROM customers;

    -- COUNT THE NUMBER OF ORDERS PLACED IN 2017

    SELECT COUNT(*) AS total_orders
    FROM orders
    WHERE YEAR(order_purchase_timestamp) = 2017;
    ```
    
### INTERMEDIATE QUERIES
* ```sql
    -- CALCULATE THE NUMBER OF ORDERS PER MONTH IN 2018 

    SELECT MONTH(order_purchase_timestamp) AS month_number,MONTHNAME(order_purchase_timestamp) AS  months, COUNT(order_id) AS order_count
    FROM orders
    WHERE YEAR(order_purchase_timestamp)  = 2018
    GROUP BY month_number,months
    ORDER BY month_number;

    -- FIND THE AVERAGE NUMBER OF PRODUCTS PER ORDER, GROUPED BY CUSTOMER CITY

    WITH count_per_order AS(
    SELECT o.order_id,o.customer_id, COUNT(oi.order_id) AS oc
    FROM orders AS o
    JOIN order_items AS oi ON o.order_id = oi.order_id
    GROUP BY o.order_id, o.customer_id)

    SELECT c.customer_city AS city, ROUND(AVG(cpo.oc),2) AS avg_orders
    FROM customers AS c
    JOIN count_per_order AS cpo ON c.customer_id = cpo.customer_id
    GROUP BY city
    ORDER BY avg_orders DESC;
  ```

### HARD QUERIES
*  ```sql
    -- CALCULATE THE MOVING AVERAGE OF ORDER VALUES FOR EACH CUSTOMER OVER THEIR ORDER HISTORY

    SELECT customer, order_date,
	       ROUND(AVG(order_value) OVER(PARTITION BY customer ORDER BY order_date
                                 ROWS BETWEEN 2 PRECEDING AND CURRENT ROW),2) AS moving_avg
    FROM (SELECT o.customer_id AS customer, o.order_purchase_timestamp AS order_date, SUM(p.payment_value) AS order_value
    FROM orders AS o
    JOIN payments AS p ON o.order_id = p.order_id
    GROUP BY customer, order_date) AS order_value_data;


    -- CALCULATE THE CUMULATIVE SALES PER MONTH FOR EACH YEAR

    SELECT year, month_name, 
           SUM(sales_value) OVER(ORDER BY year, month_number) AS cumulative_sum
    FROM (SELECT YEAR(order_purchase_timestamp) AS year, MONTH(order_purchase_timestamp)as month_number,
	       MONTHNAME(order_purchase_timestamp) AS month_name, SUM(payment_value) AS sales_value
    FROM orders AS o
    JOIN payments AS p ON o.order_id = p.order_id
    GROUP BY year,month_number,month_name
    ORDER BY year, month_number,month_name) AS sales_value_data;
    ```

-  Download SQL Queries & Schema : [ecommerce_project.sql](https://github.com/Santosh96736/e_commerce_project/blob/main/ecommer_project.sql)

## 6. Jupyter Code

-  Using Jupyter Notebook, the following steps were performed using Python, Pandas, Seaborn, and Matplotlib:

-  Loaded all CSV files and performed data cleaning.

-  Exported cleaned DataFrames like cleaned_customers.csv.

-  Used SQLAlchemy to upload cleaned data into MySQL.

-  Queried the database using Python and visualized results using Seaborn and Matplotlib.

-  Download Jupyter Code : [ecommerce_project_code.ipynb](https://github.com/Santosh96736/e_commerce_project/blob/main/ecommerce_project_code.ipynb)

## 7. Key Findings 

-  Sales Trends: Most orders were placed in 2017, peaking in November due to Black Friday sales.

-  Top Categories: 'bed_bath_table', 'health_beauty', and 'sports_leisure' generated the most revenue.

-  Customer Locations: Sao Paulo and Rio de Janeiro had the highest number of customers.

![Image](https://github.com/Santosh96736/e_commerce_project/blob/main/Top%2010%20most%20customer%20city.png)

-  Payment Preferences: 73.92% of orders were paid via credit card.

-   Growth Rate:  2017 - 12112.70%, 2018 - 20.00%
   
![Image](https://github.com/Santosh96736/e_commerce_project/blob/main/Year_Over_Yera_Growth.png)

## 8. Suggestion and Recommendation

-  Boost Customer Retention : Implement loyalty programs, follow-ups, and personalized offers. Encourage second purchases via targeted campaigns.

- Optimize Product Portfolio: Focus on top-performing categories; reevaluate or promote underperformers. Use reviews and ratings for product quality feedback.

-  Track Returns & Refunds: Integrate return/refund data to measure product satisfaction and improve services.

## 9. Contact

-  LINKEDIN : [Santosh Kumar Sahu](https://www.linkedin.com/in/santosh-kumar-sahu-data-analyst)
-  EMAIL : [santosh96736@gmail.com](santosh96736@gmail.com)


