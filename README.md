# DATA WAREHOUSE IN RETAIL AND E-COMMERCE INDUSTRY

## Executive Summary
Data warehousing is a critical step towards building a large and scalable analytics database for retail and e-commerce. A data warehouse empowers businesses with huge historical data that is accessible for analysis to generate trends that help businesses make informed decisions. This project aims to source three datasets from Kaggle: one is a relational database, another is a CSV file, and the last one is a JSON file, and uses them to generate a database that is based on the Inmon Model. In this way, the project mimics a real-life retail or e-commerce business that sources data from relational databases, flat files, and API sources. The work follows the ETL framework using Python codes to extract the data from Olist SQLite, Online Sales CSV, and Flipkart JSON, to transform it based on the data warehouse architecture created, and then load it to a newly created database. Also, the work conducts OLAP operations and visualizations on the newly created database.

## Case Study Selection
The following analysis will be based on the retail and e-commerce industry. In this sector, data analysis helps in many ways, such as monitoring revenue, tracking the most ordered products, understanding product weights, shipping costs, customer buying patterns, time series analysis of sales performance, and analyzing geographic distribution (Saini 2021). As a result, data warehousing in this industry is crucial for creating a database that can be accessed for analytics purposes.

## The Three Datasets Used

### a) Relational Database (SQL)
- Data file from relational database (Olist SQLite)  
- Google Drive Link: [https://drive.google.com/file/d/1JZTot-4VtEonFDx_I1nQCQgi6_Zs2OtK/view?usp=sharing](https://drive.google.com/file/d/1JZTot-4VtEonFDx_I1nQCQgi6_Zs2OtK/view?usp=sharing)

### b) Flat File (CSV)
- Data file from a flat file (Online Sales CSV)  
- Google Drive Link: [https://drive.google.com/file/d/1fCrK6mXFSADNJFajQcX9EQh9SnEYyF4I/view?usp=sharing](https://drive.google.com/file/d/1fCrK6mXFSADNJFajQcX9EQh9SnEYyF4I/view?usp=sharing)

### c) API Data (JSON)
- Data file from API (Flipkart JSON)  
- Google Drive Link: [https://drive.google.com/file/d/1s5PEE6coFr9Yre8-EFxyxPcwOira_-LP/view?usp=sharing](https://drive.google.com/file/d/1s5PEE6coFr9Yre8-EFxyxPcwOira_-LP/view?usp=sharing)

## Architecture Design
Using the above three datasets (Olist SQLite, Online Sales CSV, and Flipkart JSON) from Kaggle, it is possible to create a robust data warehouse architecture for a retail and commerce business, as explained below. The warehouse uses the Inmon model to structure the schema (Yessad and Labiod, 2016).

### Subject Area Determination (Conceptual Stage)
This work generates three subject areas: sales, products, and customers, and this selection matches key retail enterprise processes. The following table summarizes these areas, their source datasets, and features that will be captured when storing their data. When integrating the three datasets, Olist SQLite brings structured transactional data, and Online Sales CSV brings external sales records. At the same time, Flipkart is an API data that will be used for product catalog enrichment.

| Subject Area | Source Datasets | Key entities |
|--------------|----------------|--------------|
| Sales        | Olist SQLite, CSV | order_id, product_id, customer_id, revenue |
| Products     | Olist SQLite, Flipkart JSON | product_id, category, price, weight |
| Customers    | Olist SQLite, CSV | customer_id, zip_code, city, state |

### Logical Model (Normalized 3NF)
To ensure data clarity and integrity, a logical model is further developed to support each subject defined in the above conceptual framework (Yessad and Labiod, 2016).

| Table Name | Purpose | Features |
|------------|---------|----------|
| fact_sales | Stores sales transactions and links to dimension tables | customer_id (FK), product_id (FK), order_id, price, revenue, source, shipping_cost |
| dim_products | Stores product details | product_id (PK), product_category_name, product_weight_g, price |
| dim_customers | Stores customer details and location | customer_id (PK), geolocation_zip_code_prefix (FK), city, state |

### Physical Levels
To align with the Inmon model (Yessad and Labiod, 2016), an SQLite database (`retail_warehouse.db`) is created, and then a Python function, `initialize_database()`, is used to create three data marts: sales, products, and customers. They are the `fact_sales` table, the `dim_products` table, and the `dim_customers` table. The Python script also uses SQLite-specific tunings for faster loads.

## ETL Pipeline Implementation

### Extraction
To start with, the Python function `chunked_extract_sqlite()` is used to extract data from the Olist SQLite in batches to minimize overloading the memory. This function also accommodates those entries that have or lack rowid. In addition, this function creates pandas DataFrames in batches. Secondly, the Python function `extract_online_data()` uses `chunked_extract_sqlite()` to source batches of data from numerous Olist SQLite files. The script is also improved to remove records that have null entries in vital features such as `order_id` and `product_id`. Then the function `extract_online_sales()` is used in drawing data from the Online Sales CSV file. The function also processes this data using chunks to seamlessly extract large sizes of data without exhausting the memory. Finally, the function `extract_flipkart_products()` is run to parse the JSON file and extract product details. It also draws this data using batches and normalizes the nested JSON format by converting it into tabular format.

### Transform
The ETL pipeline uses three Python functions to transform the extracted data before loading it into the newly created database.  

1. The first function, `transform_products()`, standardizes and merges data from the three different datasets. The function imputes missing product weights using median weight per category. The script also converts non-numeric prices to numeric format. In other words, the function removes inconsistencies in extracted data by eliminating the possibility of currency symbols and commas. Also, this script bars duplicate entries.  

2. The function `transform_customers()` is applied to standardize and validate sales transactions, to join customer data with location details. The function also replaces missing city and state values with the most sensible data. The function is also modified to choose only important columns to improve its robustness.  

3. The `transform_sales()` function is applied to combine Olist SQLite extracted data with Online Sales CSV. The function also transforms all prices to numeric values and calculates prices while dropping null entries to improve data quality.

### Load Stage
In the loading stage, the function `optimized_load()` helps to seamlessly integrate extracted data and transformed data and load it efficiently into the database. It loads huge datasets in chunks to reduce memory overload. The function accesses each data batch, dynamically creates an SQL INSERT statement, and executes huge entries using `executemany()` code enhancement.  

**Final SQLite database file created by the ETL pipeline:**  
[https://drive.google.com/file/d/1mTQqNqgwjBW-40n8AHFhMcAA-81CPE9a/view?usp=sharing](https://drive.google.com/file/d/1mTQqNqgwjBW-40n8AHFhMcAA-81CPE9a/view?usp=sharing)

## OLAP Operations and Results

### Roll-up
After the large database was created by the ETL pipeline, it is possible now to conduct OLAP operations efficiently to derive business insights (Luoma, 2024; Yessad and Labiod, 2016). For instance, a roll-up function, `revenue_by_category()`, is used to aggregate the revenue by product category. The results show each product category with its respective revenue generated from sales in that category, the total number of unique orders in that category, and the average revenue per order. The query orders the results by revenue in descending order while limiting the output to the top 10 categories. From the results, `beleze_saude` (Beauty and Health) is the category that generated the hugest total revenue of 1,258,681.34. It was ordered 8,836 times, which indicates that it attracted many buyers. However, its average order value of 130 was lower than other categories. Also, `relogios_presentes` (watches and gifts) and `cama_mesa_banho` (Bedding and bath) were in the top three most revenue-generating product categories.

### Drill-Down
A Drill-Down Python function, `drilldown_category()`, is used to query the database to explore the top 5 profitable products in a certain product category. For instance, when the “cool_stuff” product category is keyed in the function, it generates the most product categories in the “cool_stuff” category. In this context, the top 3 most profitable products in this category are:  

- `5f504b3a1c75b73d6151be81eb05bdc9`: $37,733.9  
- `165f86fe8b799a708a20ee4ba125c289`: $17,820.91  
- `601a360bd2a916ecef0e88de72a6531a`: $15,159.81  

Also, this drill-down function shows the count of unique orders generated by each product.

### Visualization and Insights
It is also possible to visualize the results of OLAP operations (Luoma, 2024). In the Jupyter notebook file (Program file), you will see the graphs that were used to visualize the results of Roll-Up and Drill-Down.

## Conclusion
The above work has successfully demonstrated how the Inmon Model can be used in generating data warehouses in the retail and e-commerce sectors. Using three datasets (Olist SQLite, Online Sales CSV, and Flipkart JSON) from Kaggle, it was possible to generate a warehouse with three dimensions (sales, products, and customers), and this selection matches key retail enterprise processes. The warehouse further developed a logical model to support each subject. To complete the physical stage of the Inmon model, an SQLite database (`retail_warehouse.db`) is created using the Python function and further manipulated to develop the three data marts that are enclosed in the `fact_sales` table, `dim_products` table, and `dim_customers` table. The ETL pipeline successfully managed to extract and consolidate raw sales data from multiple sources (Olist, Flipkart, and online sales) into a structured format. It then transformed and cleaned the data before loading it into the optimized SQLite database (`retail_warehouse.db`) to enable efficient analysis and reporting. OLAP operations conducted on this SQLite database show that `beleze_saude` (Beauty and Health) is the category that generated the hugest total revenue of 1,258,681.34. It was ordered 8,836 times, which indicates that it attracted many buyers. In addition, the top 3 most profitable products in this category are:  

- `5f504b3a1c75b73d6151be81eb05bdc9`: $37,733.9  
- `165f86fe8b799a708a20ee4ba125c289`: $17,820.91  
- `601a360bd2a916ecef0e88de72a6531a`: $15,159.81  

As observed from the interactive dashboard, it can be seen that most of the revenue came from the Olist SQLite database. Also, the interactive dashboard summarizes products with the most sales while simultaneously showing their labels, their categories, their average, and their source. From the dashboard, the product with the highest average order value is “pcs.”

## References
- Luoma, S., 2024. Data warehousing in Finnish municipalities: case city of Espoo.  
- Saini, A., 2021. The role of data warehousing in the infrastructure of E-commerce. IITM Journal of Management and IT, 12(2), pp.39-47.  
- Yessad, L. and Labiod, A., 2016, November. Comparative study of data warehouses modeling approaches: Inmon, Kimball and Data Vault. In 2016 International Conference on System Reliability and Science (ICSRS) (pp. 95-99). IEEE.

