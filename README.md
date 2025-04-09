# e_commerce_project

## TABLE CONTENTS
1. [PROJECT OVERVIEW](PROJECT-OVERVIEW)
2. [PROJECT OBJECTIVE](PROJECT-OBJECTIVE)
3. [DATABASE SCHEMA](DATABASE-SCHEMA)
4. [SQL QUERIES](SQL-QUERIES)
5. [JUPYTER CODE](JUPYTER-CODE)
6. [KEY FINDINGS](KEY-FINDINGS)
7. [REPOSITORY DETAILS](REPOSITORY-DETAILS)

## 1. PROJECT OVERVIEW
*   This project involves deep analysis of e_commerce dataset to derive meaningful
    business insights using python, MySQL, and data visualization tools like Seaborn and matplotlib.


## 2. PROJECT OBJECTIVE
*    **Clean data** : Handle missing value, duplicates, and formatting using Python.
*    **Upload data** : Load the cleaned dataset into MySQL 
*    **Query data** : Use MySQL and Python for analysis
*    **Visualize data** : Create graphs and charts using Seaborn and Matplotlib

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

## 5. JUPYTER CODE

### DATA CLEANING

```python
# Cleaning Orders & Customers Data
- Remove duplicates
- Handle missing values
- Convert date formats

import pandas as pd

# Load Dataset
df_orders = pd.read_csv("orders.csv")
df_customers = pd.read_csv("customers.csv")

# Convert relevant columns to datetime format

df_orders["order_purchase_timestamp"] = pd.to_datetime(df_orders["order_purchase_timestamp"])
df_orders["order_approved_at"] = pd.to_datetime(df_orders["order_approved_at"])
df_orders["order_delivered_carrier_date"] = pd.to_datetime(df_orders["order_delivered_carrier_date"])
df_orders["order_delivered_customer_date"] = pd.to_datetime(df_orders["order_delivered_customer_date"])
df_orders["order_estimated_delivery_date"] = pd.to_datetime(df_orders["order_estimated_delivery_date"])

# Function to check dataset summary
def check_dataset(df, name):
    print("\n" + "=" * 60)
    print(f"Dataset Summary: {name}")
    print("=" * 60)

    print(f"\n Missing Values:\n{df.isnull().sum()}")
    print(f"\n Duplicate Rows: {df.duplicated().sum()}")
    
    print("\n Dataset Info:")
    df.info()
    
    print("\n Statistical Summary:")
    print(df.describe())


# Check orders dataset
check_dataset(df_orders, "Orders")

# Checking Null Values where order_status == "shipped"
shipped_missing = df_orders[df_orders["order_status"] == "shipped"].isnull().sum()
print("\nMissing values in 'shipped' orders:")
print(shipped_missing)

# Fill missing "order_delivered_customer_date" with "order_estimated_delivery_date" only for shipped orders
df_orders.loc[(df_orders["order_status"] == "shipped") & 
              (df_orders["order_delivered_customer_date"].isnull()), 
              "order_delivered_customer_date"] = df_orders["order_estimated_delivery_date"]

# Re-check missing values to confirm they are correctly filled
shipped_missing_after = df_orders[df_orders["order_status"] == "shipped"].isnull().sum()
print("\nMissing values in 'shipped' orders after filling:")
print(shipped_missing_after)

# Upload cleaned "df_orders" into "cleaned_orders.csv"
df_orders.to_csv("cleaned_orders.csv", index= False)

print("\n" + "-" * 60)

# Check customers dataset
check_dataset(df_customers, "Customers")

# Upload cleaned "df_customers" into "cleaned_customer.csv"
df_customers.to_csv("cleaned_customers.csv", index= False)

print("\n" + "-" * 60)
```

### UPLOADING DATA INTO MYSQL
```python
import pandas as pd
import mysql.connector
import os
from dotenv import load_dotenv

load_dotenv()

# List of CSV files and their corresponding table names
csv_files = [
    ('cleaned_customers.csv', 'customers'),
    ('cleaned_orders.csv', 'orders'),
    ('cleaned_sellers.csv', 'sellers'),
    ('cleaned_products.csv', 'products'),
    ('cleaned_order_items.csv', 'order_items'),
    ('cleaned_payments.csv', 'payments'),
    ('cleaned_geolocation.csv', 'geolocation'),
    ('cleaned_order_reviews.csv', 'order_reviews')
]

# Connect to the MySQL database
conn = mysql.connector.connect(
    host= os.getenv("DB_HOST"),
    user= os.getenv("DB_USER"),
    password= os.getenv("DB_PASSWORD"),
    database= os.getenv("DB_NAME")
)
cursor = conn.cursor()

# Folder containing the CSV files
folder_path = 'D:/E_Commerce_Project'

# Function to determine SQL data type
def get_sql_type(column_name, dtype):
    if "geolocation_lat" in column_name or "geolocation_lng" in column_name:
        return 'DECIMAL(18,15)'  # High precision for latitude & longitude
    elif pd.api.types.is_integer_dtype(dtype):
        return 'INT'
    elif pd.api.types.is_float_dtype(dtype):
        return 'DECIMAL(10,2)'  # General decimal format for prices, weights, etc.
    elif pd.api.types.is_datetime64_any_dtype(dtype):
        return 'DATETIME'
    else:
        return 'VARCHAR(50)'  # Default VARCHAR

# Process each CSV file
for csv_file, table_name in csv_files:
    file_path = os.path.join(folder_path, csv_file)
    
    # Read the CSV file into a pandas DataFrame
    df = pd.read_csv(file_path)
    
    # Replace NaN with None to handle SQL NULL values
    df = df.where(pd.notnull(df), None)

    # Debugging: Check for NaN values
    print(f"Processing {csv_file}")
    print(f"NaN values before replacement:\n{df.isnull().sum()}\n")

    # Clean column names to match MySQL naming conventions
    df.columns = [col.replace(' ', '_').replace('-', '_').replace('.', '_') for col in df.columns]

    # Convert ENUM columns to lowercase and validate values
    if 'order_status' in df.columns:
        df['order_status'] = df['order_status'].str.lower().str.strip()
        valid_status = {'delivered', 'invoiced', 'shipped', 'processing', 'unavailable', 'canceled', 'created', 'approved'}
        df = df[df['order_status'].isin(valid_status)]  # Remove invalid values

    if 'payment_type' in df.columns:
        df['payment_type'] = df['payment_type'].str.lower().str.strip()
        valid_payment = {'credit_card', 'upi', 'voucher', 'debit_card', 'not_defined'}
        df = df[df['payment_type'].isin(valid_payment)]

    # Insert DataFrame data into the MySQL table
    for _, row in df.iterrows():
        values = tuple(None if pd.isna(x) else x for x in row)
        sql = f"INSERT INTO `{table_name}` ({', '.join(['`' + col + '`' for col in df.columns])}) VALUES ({', '.join(['%s'] * len(row))})"
        
        try:
            cursor.execute(sql, values)
        except mysql.connector.Error as err:
            print(f"Error inserting into {table_name}: {err}")
    
    # Commit the transaction for the current CSV file
    conn.commit()

# Close the connection
conn.close()
print("✅ Data Upload Completed Successfully!")
```

### CONNECTING TO MySQL
```python
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
import mysql.connector
import os
from dotenv import load_dotenv

load_dotenv()

mydb = mysql.connector.connect(host = os.getenv("DB_HOST"),
                               user = os.getenv("DB_USER"),
                               password = os.getenv("DB_PASSWORD"),
                               database = os.getenv("DB_NAME"))

cur = mydb.cursor()
```

### IDENTIFY THE TOP 3 CUSTOMERS WHO SPENT THE MOST MONEY IN EACH YEAR
```python
query = """WITH customer_rank_data AS (SELECT year, customer, amount_spent,
       DENSE_RANK() OVER(PARTITION BY year ORDER BY amount_spent DESC) AS customer_rank
FROM (SELECT YEAR(o.order_purchase_timestamp) AS year,o.customer_id AS customer, SUM(p.payment_value) AS amount_spent
FROM orders AS o
JOIN payments AS p ON o.order_id = p.order_id
GROUP BY year,o.customer_id
ORDER BY year) AS speding_data)

SELECT year, customer, amount_spent
FROM customer_rank_data
WHERE customer_rank <= 3;"""

cur.execute(query)

data = cur.fetchall()

df = pd.DataFrame(data, columns = ["year","customer","amount spent"])

plt.figure(figsize = (10,8))

sns.set_style("dark")

ax = sns.barplot(data = df, x = "customer", y = "amount spent", hue = "year", palette= "viridis")

for container in ax.containers:
    plt.bar_label(container, fmt = "%d", label_type= "edge", weight = "bold")

plt.xticks(rotation = 90)

plt.xlabel("Customer ID", fontsize = 15)
plt.ylabel("Amount Spent By Customer", fontsize = 15)
plt.title("Top 3 spending customer per year", fontsize = 20, fontweight = "bold")

plt.grid(axis = "y", linestyle = "--", alpha = 0.7)

plt.show()
```
![top 3 spending customer](https://raw.githubusercontent.com/Santosh96736/e_commerce_project/refs/heads/main/top%203%20spending%20customers.png)

## 6. KEY FINDINGS 
*  i. **Orders placed in 2017** : 45101
*  ii. **Percentage paid via credit card** : 73.92%
*  iii. **Correlation between Price-Purchases** : -0.10
*  iv. **Year-Over-Year Growth Rate** : 12112.70% (2017), 20.00% (2018)
*  v. **Retetion Rate** : 0

## 7. REPOSITORY DETAILS
*  i. **Repository Name** : e_commerce_project
*  ii. **Download Dataset** : [e-Commerce (Target) Sales Dataset - Kaggle](https://www.kaggle.com/datasets/devarajv88/target-dataset)
*  ⚠️ Note: The file order_reviews.csv has been removed from the dataset by the original author. If anyone wanted to the file can download from my github repository.
*  iii. **Download Missing File** : [order_review.csv](https://github.com/Santosh96736/e_commerce_project/blob/main/order_reviews.csv)
*  iv. **Contact** : [santosh kumar sahu](santosh96736@gmail.com)

