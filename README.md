# e_commerce_project

## TABLE CONTENTA
1. [PROJECT OVERVIEW](PROJECT-OVERVIEW)
2. [PROJECT OBJECTIVE](PROJECT-OBJECTIVE)
3. [DATABASE SCHEMA](DATABASE-SCHEMA)
4. [SQL QUERIES](SQL-QUERIES)
5. [JUPYTER CODE](JUPYTER-CODE)
6. [KEY FINDINGS](KEY-FINDINGS)
7. [REPOSITORY DETAILS](REPOSITORY-DETAILS)

## 1. PROJECT OVERVIEW
*   This project invovles deep analysis of e_commerce_datase to derieve valueable
    business insights using python and MySQL.


## 2. PROJECT OBJECTIVE
*    **Clean data** : Use python to clean data
*    **Uplode data** : Use python code to upload data into MySQL 
*    **Query data** : Query data using MySQL and python
*    **Visualize data** : Visualize data using python seaborn and matplotlib

## 3. DATABASE SCHEMA
>  |**Table Name**|  **Column Name**|
>  |--------------|-----------------------------------------------------------------------------------------------------------------------|
>  | **Customer** | `customer_id`, `customer_unique_id` , `customer_zip_code_prefix`, `customer_city`, `customer_state`|
>  | **Geolocation** | `geolocation_zip_code_prefix` , `geolocation_lat`, `geolocation_lng`, `geolocation_city`, `geolocation_state`|
>  | **Order_Items** | `order_id`, `order_item_id`, `product_id`, `seller_id`, `shipping_limit_date`, `price`, `freight_value`|
>  | **Order_Reviews** | `review_id`, `order_id`, `review_score`, `review_comment_title`, `review_creation_date`, `review_answer_timestamp`|
>  | **Orders** | `order_id`, `customer_id`, `order_status`, `order_purchase_timestamp`, `order_approved_at`,|  
>  |            |   `order_delivered_carrier_date`, `order_delivered_customer_date`, `order_estimated_delivery_date`|
>  | **Payments** | `order_id`, `payment_sequential`, `payment_type`, `payment_installments`, `payment_value`|
>  | **Products** | `product_id`, `product_category`, `product_name_length`, `product_description_length`, `product_photos_qty`,
>  |              | `product_weight_g`, `product_length_cm`, `product_height_cm`, `product_width_cm`|
>  | **Sellers** | `seller_id`, `seller_zip_code_prefix`, `seller_city`, `seller_state`

## 4. SQL QUERIES

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
    
    
    CREATE TABLE sellers(
    seller_id VARCHAR(35),
    seller_zip_code_prefix INT,
    seller_city VARCHAR(40),
    seller_state VARCHAR(5),
    PRIMARY KEY (seller_id)
    );


    CREATE TABLE customers(
    customer_id VARCHAR(35),
    customer_unique_id VARCHAR(35),
    customer_zip_code_prefix INT,
    customer_city VARCHAR(35),
    customer_state VARCHAR(5),
    PRIMARY KEY (customer_id)
    );


    CREATE TABLE products(
    product_id VARCHAR(35),
    product_category VARCHAR(50),
    product_name_length INT,
    product_description_length INT,
    product_photos_qty INT,
    product_weight_g INT,
    product_length_cm INT,
    product_height_cm INT,
    product_width_cm INT,
    PRIMARY KEY (product_id)
    );

    CREATE TABLE orders(
    order_id VARCHAR(35),
    customer_id VARCHAR(35),
    order_status ENUM('delivered', 'invoiced', 'shipped', 'processing', 'unavailable', 'canceled', 'created', 'approved'),
    order_purchase_timestamp DATETIME,
    order_approved_at DATETIME,
    order_delivered_carrier_date DATETIME,
    order_delivered_customer_date DATETIME,
    order_estimated_delivery_date DATETIME,
    PRIMARY KEY (order_id),
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
    );


    CREATE TABLE order_items(
    order_id VARCHAR(35),
    order_item_id INT,
    product_id VARCHAR(35),
    seller_id VARCHAR(35),
    shipping_limit_date DATETIME,
    price DECIMAL(10,2),
    freight_value DECIMAL(10,2),
    PRIMARY KEY (order_id, order_item_id),
    FOREIGN KEY (order_id) REFERENCES orders(order_id),
    FOREIGN KEY (product_id) REFERENCES products(product_id),
    FOREIGN KEY (seller_id) REFERENCES sellers(seller_id)
    );


    CREATE TABLE order_reviews(
    review_id VARCHAR(35),                 
    order_id VARCHAR(35),                 
    review_score INT CHECK(review_score BETWEEN 1 AND 5),             
    review_comment_title VARCHAR(45),
    review_creation_date DATETIME,
    review_answer_timestamp DATETIME,
    PRIMARY KEY (review_id),
    FOREIGN KEY (order_id) REFERENCES orders(order_id)
    );


    CREATE TABLE payments(
    order_id VARCHAR(35),
    payment_sequential INT,
    payment_type ENUM('credit_card', 'UPI', 'voucher', 'debit_card', 'not_defined'),
    payment_installments INT,
    payment_value DECIMAL(10,2),
    PRIMARY KEY (order_id, payment_sequential),
    FOREIGN KEY (order_id) REFERENCES orders(order_id)
    );
    ```

### EASY 

*   ```sql
    -- LIST ALL UNIQUE CITIES WHERE CUSTOMERS ARE LOCATED

    SELECT DISTINCT customer_city
    FROM customers;

    -- COUNT THE NUMBER OF ORDERS PLACED IN 2017

    SELECT COUNT(*) AS total_orders
    FROM orders
    WHERE YEAR(order_purchase_timestamp) = 2017;

    -- FIND TOTAL SALES PER CATEGORY

    WITH payment_data AS (SELECT oi.product_id, SUM(p.payment_value) AS sales
    FROM payments AS p
    JOIN order_items AS oi ON p.order_id = oi.order_id
    GROUP BY oi.product_id)

    SELECT LOWER(product_category) AS category, SUM(sales) AS sales
    FROM payment_data AS pd
    JOIN products AS p ON p.product_id = pd.product_id
    GROUP BY product_category;


    -- CALCULATE THE PERCENTAGE OF ORDERS THAT WERE PAID THROUGH CREDIT-CARD

    SELECT CONCAT(ROUND(((SELECT COUNT(payment_type) FROM payments WHERE payment_type = "credit_card")/ COUNT(*)) * 100,2),"%") AS percentage
    FROM payments;


    -- CALCULATE NUMBER OF CUSTOMERS IN EACH CITY

    SELECT customer_city, COUNT(customer_id) AS total_customer
    FROM customers
    GROUP BY customer_city;
    ```
    
### INTERMEDIATE
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

    -- CALCULATE THE PERCENTAGE OF TOTAL REVENUE CONTRIBUTED BY EACH PRODUCT CATEGORY

    WITH payment_data AS (SELECT oi.product_id, SUM(p.payment_value) AS sales
    FROM payments AS p
    JOIN order_items AS oi ON p.order_id = oi.order_id
    GROUP BY oi.product_id)

    SELECT LOWER(product_category) AS category, 
         ROUND((SUM(sales) / (SELECT SUM(payment_value) FROM payments)) * 100,2) AS sales_percentage
    FROM payment_data AS pd
    JOIN products AS p ON p.product_id = pd.product_id
    GROUP BY category
    ORDER BY sales_percentage DESC;


    -- IDENTIFY THE CORRELATION BETWEEN PRODUCT PRICE AND THE NUMBER OF TIMES A PRODUCT HAS BEEN PURCHASED

    SELECT LOWER(p.product_category) AS category, COUNT(oi.order_id) AS order_count, ROUND(AVG(oi.price),2) AS price
    FROM products AS p
    JOIN order_items AS oi ON p.product_id = oi.product_id
    GROUP BY category;

    -- CALCULATE THE TOAL REVENUE GENERATED BY EACH SELLER, AND RANK THEM BY REVENUE

    SELECT seller_id, revenue,
           DENSE_RANK() OVER(ORDER BY revenue DESC) AS seller_rank
    FROM (SELECT oi.seller_id, SUM(p.payment_value) AS revenue
    FROM order_items AS oi
    JOIN payments AS p ON oi.order_id = p.order_id
    GROUP BY oi.seller_id) AS revenue_data
    LIMIT 10;
  ```

### HARD
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


    -- CALCULATE THE YEAR-OVER-YEAR GROWTH RATE OF TOTAL SALES

     SELECT year, 
		    ROUND(((sales_value - LAG(sales_value) OVER(ORDER BY year))/ LAG(sales_value) OVER(ORDER BY year)) * 100,2) AS yoy_growth
     FROM(SELECT YEAR(o.order_purchase_timestamp) AS year, SUM(p.payment_value) AS sales_value
     FROM orders AS o
     JOIN payments AS p ON o.order_id = p.order_id
     GROUP BY year) AS sales_value_data;
 
    -- CALCULATE THE RETENTION RATE OF CUSTOMERS, DEFINED AS THE PERCENTAGE OF OF CUSTOMER WHO MAKE ANOTHER PUCHASE 
       WITHIN 6 MONTHS OF THEIR FIRST PURCHASE 

    WITH first_order_customer AS 
    (SELECT c.customer_id, MIN(o.order_purchase_timestamp) AS first_order
    FROM customers AS c
    JOIN orders AS o ON c.customer_id = o.customer_id
    GROUP BY c.customer_id),

    repeat_order_customer AS 
    (SELECT o.customer_id
    FROM orders AS o
    JOIN first_order_customer AS f ON o.customer_id = f.customer_id
                               AND o.order_purchase_timestamp > f.first_order
                               AND o.order_purchase_timestamp <= DATE_ADD(f.first_order, INTERVAL 6 MONTH))

    SELECT ROUND((COUNT(DISTINCT r.customer_id) / COUNT(DISTINCT f.customer_id)) * 100,2) AS retention_rate
    FROM repeat_order_customer AS r
    JOIN first_order_customer AS f ON r.customer_id = f.customer_id;


    -- IDENTIFY THE TOP 3 CUSTOMERS WHO SPENT THE MOST MONEY IN EACH YEAR

    WITH customer_rank_data AS (SELECT year, customer, amount_spent,
           DENSE_RANK() OVER(PARTITION BY year ORDER BY amount_spent DESC) AS customer_rank
    FROM (SELECT YEAR(o.order_purchase_timestamp) AS year,o.customer_id AS customer, SUM(p.payment_value) AS amount_spent
    FROM orders AS o
    JOIN payments AS p ON o.order_id = p.order_id
    GROUP BY year,o.customer_id
    ORDER BY year) AS speding_data)

    SELECT year, customer, amount_spent
    FROM customer_rank_data
    WHERE customer_rank <= 3;
    ```

