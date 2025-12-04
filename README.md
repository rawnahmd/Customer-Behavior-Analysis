# Customer-Behavior-Analysis
Data Analysis project to transform sales and customer data into actionable insights with SQL, Python, and Power BI dashboard

📊 Customer Behavior Analytics
End-to-End Data Project Using Python, SQL, and Power BI

This project demonstrates a complete analytics workflow—starting from raw data, moving through Python-based cleaning and feature engineering, applying SQL for business analysis, and finally presenting insights through an interactive Power BI dashboard.

It reflects the ability to transform messy, real-world data into clean, reliable insights for business decision-making.

```
customer-behavior-analytics/
│
├── customer_behavior.sql
├── customer_shopping.ipynb
├── Customer_Behavior_Dashboard.pbix
└── README.md
```

Python Data Cleaning & Feature Engineering

(customer_shopping.ipynb)

The dataset initially contained missing values, inconsistent formats, mixed casing, and unclear categorical labels.
The Python pipeline prepares the data for structured SQL analysis and dashboarding.

✔ Load & Inspect Data

The dataset is loaded into a Pandas DataFrame, checked for missing values, inconsistencies, and formatting issues.

✔ Missing Value Treatment (Mean vs Median Strategy)

Mean → used for normally distributed features

Median → used for skewed features or when outliers are present

Code used:
```python
df['Review Rating'] = df.groupby('Category')['Review Rating'].transform(
    lambda x: x.fillna(x.median())
)
```

✔ Standardizing Column Names
```python
df.columns = df.columns.str.lower()
df.columns = df.columns.str.replace(' ', '_')
df = df.rename(columns={'purchase_amount_(usd)': 'purphase_amount'})
```
✔ Creating Age Groups (qcut)
```python
labels = ['Young Adult', 'Adult', 'Middle_aged', 'Senior']
df['age_group'] = pd.qcut(df['age'], q=4, labels=labels)
```
✔ Mapping Purchase Frequency to Days
```python
frequency_days = {
    'Fortnightly': 14,
    'Weekly': 7,
    'Bi-Weekly': 14,
    'Monthly': 30,
    'Every 3 Months': 90, 
    'Quarterly': 90,
    'Annually': 365
}

df['purphase_frequency_days'] = df['frequency_of_purchases'].map(frequency_days)
```
✔ Loading Clean Data into MySQL Database
```python
import pandas as pd
from sqlalchemy import create_engine
import mysql.connector
```
# Define connection parameters
```python
host = "localhost"
port = 3306
user = "root"
password = "hg1a23ce"
database = "customer_behavior"
```
# MySQL connector connection
```python
conn = mysql.connector.connect(
    host=host,
    port=port,
    user=user,
    password=password,
    database=database
)
```
# SQLAlchemy engine
```python
engine = create_engine(f"mysql+pymysql://{user}:{password}@{host}:{port}/{database}")

# Write DataFrame to MySQL
df.to_sql("customer", engine, if_exists="replace", index=False)

# Read back sample
df_mysql = pd.read_sql("SELECT * FROM customer LIMIT 5;", engine)
print(df_mysql)
```
2️⃣ SQL Business Analysis

(customer_behavior.sql)

Below are the SQL queries used to extract insights from the cleaned dataset.

✔ Total Revenue by Gender
```sql
SELECT gender, SUM(purphase_amount) AS revenue
FROM customer
GROUP BY gender;
```
✔ Customers Who Used a Discount and Spent More Than Average
```sql
SELECT customer_id, purphase_amount
FROM customer
WHERE discount_applied = 'Yes'
  AND purphase_amount >= (SELECT AVG(purphase_amount) FROM customer);
```
✔ Top 5 Products by Review Rating
```sql
SELECT item_purchased, ROUND(AVG(review_rating),2) AS average_product_rating
FROM customer
GROUP BY item_purchased
ORDER BY average_product_rating DESC
LIMIT 5;
```
✔ Compare Average Spend by Shipping Type
```sql
SELECT shipping_type, ROUND(AVG(purphase_amount),2) AS avg_purchase_amount
FROM customer
WHERE shipping_type IN ('Standard','Express')
GROUP BY shipping_type;
```
✔ Subscribers vs Non-Subscribers
```sql
SELECT subscription_status,
       COUNT(customer_id) AS total_customers,
       ROUND(AVG(purphase_amount),2) AS avg_spend,
       ROUND(SUM(purphase_amount),2) AS total_revenue
FROM customer
GROUP BY subscription_status
ORDER BY total_revenue DESC, avg_spend DESC;
```
✔ Top 5 Products with Highest Discount Usage
```sql
SELECT item_purchased,
       ROUND(100.0 * SUM(CASE WHEN discount_applied = 'Yes' THEN 1 ELSE 0 END) / COUNT(*),2) AS discount_rate
FROM customer
GROUP BY item_purchased
ORDER BY discount_rate DESC
LIMIT 5;
```

✔ Customer Segmentation (New, Returning, Loyal)
```sql
WITH customer_type AS (
    SELECT customer_id, previous_purchases,
           CASE 
               WHEN previous_purchases = 1 THEN 'New'
               WHEN previous_purchases BETWEEN 2 AND 10 THEN 'Returning'
               ELSE 'Loyal'
           END AS customer_segment
    FROM customer
)
SELECT customer_segment, COUNT(*) AS number_of_customers
FROM customer_type
GROUP BY customer_segment;
```
✔ Top 3 Most Purchased Products by Category
```sql
WITH item_counts AS (
    SELECT category,
           item_purchased,
           COUNT(customer_id) AS total_orders,
           ROW_NUMBER() OVER (PARTITION BY category ORDER BY COUNT(customer_id) DESC) AS item_rank
    FROM customer
    GROUP BY category, item_purchased
)
SELECT item_rank, category, item_purchased, total_orders
FROM item_counts
WHERE item_rank <= 3;
```
✔ Repeat Buyers by Subscription Status
```sql
SELECT subscription_status,
       COUNT(customer_id) AS repeat_buyers
FROM customer
WHERE previous_purchases > 5
GROUP BY subscription_status;
```
✔ Revenue by Age Group
```sql
SELECT age_group,
       SUM(purphase_amount) AS total_revenue
FROM customer
GROUP BY age_group
ORDER BY total_revenue DESC;
```
3️⃣ Power BI Dashboard

(Customer_Behavior_Dashboard.pbix)

Dashboard Name: Customer Review Dashboard

The dashboard summarizes insights from SQL and Python using interactive visualizations.

⭐ KPIs

3.9K customers

$59.76 average purchase amount

3.75 average review rating

📊 Visuals

Percent of Subscribers

Revenue by Category

Sales by Category

Revenue by Age Group

Sales by Age Group

🎚 Filters

Subscription Status

Gender

Category

Shipping Type

📘 About the Dataset

Includes customer demographics, purchasing behavior, subscription status, review ratings, product categories, and shipping preferences.

🚀 Key Takeaways

Complete end-to-end pipeline: Python → SQL → Power BI

Strong data cleaning and feature engineering

Business-driven SQL insights

Professional BI dashboard summarizing findings
