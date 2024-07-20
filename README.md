# E-commerce-Data-Analysis-Project
This project aims to analyze e-commerce data to derive meaningful insights about customer behavior, sales trends, and product performance. We utilize Python, MySQL, and various data visualization libraries to perform the analysis.






# E-commerce Data Analysis

This project involves a comprehensive analysis of an e-commerce database using Python, Pandas, MySQL, and various data visualization tools like Matplotlib and Seaborn. The analysis includes customer demographics, order trends, sales performance, and product popularity among other insights.

## Table of Contents
- [Installation](#installation)
- [Database Connection](#database-connection)
- [Analysis and Insights](#analysis-and-insights)
  - [Unique Cities](#unique-cities)
  - [Orders in 2017](#orders-in-2017)
  - [Sales per Category](#sales-per-category)
  - [Installment Payments](#installment-payments)
  - [Customers by State](#customers-by-state)
  - [Orders per Month in 2018](#orders-per-month-in-2018)
  - [Average Products per Order by City](#average-products-per-order-by-city)
  - [Revenue Percentage per Category](#revenue-percentage-per-category)
  - [Correlation Between Product Price and Order Count](#correlation-between-product-price-and-order-count)
  - [Revenue by Seller](#revenue-by-seller)
  - [Moving Average of Order Values](#moving-average-of-order-values)
  - [Cumulative Sales](#cumulative-sales)
  - [Year-over-Year Growth](#year-over-year-growth)
  - [Customer Retention Rate](#customer-retention-rate)
  - [Top Customers by Year](#top-customers-by-year)
- [Conclusion](#conclusion)

## Installation

1. Clone the repository:
    ```sh
    git clone https://github.com/yourusername/ecommerce-analysis.git
    ```

2. Navigate to the project directory:
    ```sh
    cd ecommerce-analysis
    ```

3. Install the required dependencies:
    ```sh
    pip install -r requirements.txt
    ```

## Database Connection

Ensure you have MySQL installed and running. Update the database connection details in your script accordingly:

```python
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
import mysql.connector
```


```python
db = mysql.connector.connect(
    host='localhost',
    user='root',
    password='Akshay@123',
    database='ecomerce'
)
```

# Creating a cursor object to interact with the database
```python
cur = db.cursor()
```
## Data Ingestion Script

The data_ingestion.py script reads each CSV file, processes it, and inserts the data into the corresponding MySQL tables. The script handles various data types and ensures that NaN values are appropriately replaced with SQL NULL.

Here is a brief overview of the script:


```python
import pandas as pd
import mysql.connector
import os

# Connect to the MySQL database
conn = mysql.connector.connect(
    host='localhost',
    user='root',
    password='Akshay@123',
    database='ecomerce'
)
cursor = conn.cursor()

# Folder containing the CSV files
folder_path = 'D:/Ecomerce project full data'

def get_sql_type(dtype):
    if pd.api.types.is_integer_dtype(dtype):
        return 'INT'
    elif pd.api.types.is_float_dtype(dtype):
        return 'FLOAT'
    elif pd.api.types.is_bool_dtype(dtype):
        return 'BOOLEAN'
    elif pd.api.types.is_datetime64_any_dtype(dtype):
        return 'DATETIME'
    else:
        return 'TEXT'

for csv_file, table_name in csv_files:
    file_path = os.path.join(folder_path, csv_file)
    
    # Read the CSV file into a pandas DataFrame
    df = pd.read_csv(file_path)
    
    # Replace NaN with None to handle SQL NULL
    df = df.where(pd.notnull(df), None)
    
    # Debugging: Check for NaN values
    print(f"Processing {csv_file}")
    print(f"NaN values before replacement:\n{df.isnull().sum()}\n")

    # Clean column names
    df.columns = [col.replace(' ', '_').replace('-', '_').replace('.', '_') for col in df.columns]

    # Generate the CREATE TABLE statement with appropriate data types
    columns = ', '.join([f'`{col}` {get_sql_type(df[col].dtype)}' for col in df.columns])
    create_table_query = f'CREATE TABLE IF NOT EXISTS `{table_name}` ({columns})'
    cursor.execute(create_table_query)

    # Insert DataFrame data into the MySQL table
    for _, row in df.iterrows():
        # Convert row to tuple and handle NaN/None explicitly
        values = tuple(None if pd.isna(x) else x for x in row)
        sql = f"INSERT INTO `{table_name}` ({', '.join(['`' + col + '`' for col in df.columns])}) VALUES ({', '.join(['%s'] * len(row))})"
        cursor.execute(sql, values)

    # Commit the transaction for the current CSV file
    conn.commit()

# Close the connection
conn.close()

```
