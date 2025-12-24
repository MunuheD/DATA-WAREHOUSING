# Data Warehouse in Retail and E-commerce Industry

## Executive Summary
Data warehousing is a critical step towards building a large and scalable analytics database for retail and e-commerce. A data warehouse empowers businesses with huge historical data that is accessible for analysis to generate trends that help businesses make informed decisions. This project aims to source three datasets from Kaggle: one is a relational database, another is a CSV file, and the last one is a JSON file, and uses them to generate a database that is based on the Inmon Model. In this way, the project mimics a real-life retail or e-commerce business that sources data from relational databases, flat files, and API sources. The work follows the ETL framework using Python codes to extract the data from Olist SQLite, Online Sales CSV, and Flipkart JSON, to transform it based on the data warehouse architecture created, and then load it to a newly created database. Also, the work conducts OLAP operations and visualizations on the newly created database.

## 📊 Project Overview
- **Industry**: Retail & E-commerce
- **Architecture**: Inmon Model
- **Database**: SQLite (`retail_warehouse.db`)
- **ETL Pipeline**: Python-based
- **Data Sources**: Olist SQLite, Online Sales CSV, Flipkart JSON
**Datasets Used:**


a) Relational Database (SQL):
- Data file from relational database 
- Link: https://drive.google.com/file/d/1JZTot-4VtEonFDx_I1nQCQgi6_Zs2OtK/view?usp=sharing

b) Flat File (CSV):
- Data file from a flat file 
- Link: https://drive.google.com/file/d/1fCrK6mXFSADNJFajQcX9EQh9SnEYyF4I/view?usp=sharing

c) API Data (JSON):
- Data file from API 
- Link: https://drive.google.com/file/d/1s5PEE6coFr9Yre8-EFxyxPcwOira_-LP/view?usp=sharing

## 🏗️ Architecture Design

### Subject Area Determination (Conceptual Stage)
This work generates three subject areas: sales, products, and customers, matching key retail enterprise processes.

| Subject Area | Source Datasets | Key Entities |
|-------------|----------------|--------------|
| Sales | Olist SQLite, CSV | order_id, product_id, customer_id, revenue |
| Products | Olist SQLite, Flipkart JSON | product_id, category, price, weight |
| Customers | Olist SQLite, CSV | customer_id, zip_code, city, state |

### Logical Model (Normalized 3NF)
| Table Name | Purpose | Features |
|------------|---------|----------|
| `fact_sales` | Stores sales transactions | customer_id (FK), product_id (FK), order_id, price, revenue, source, shipping_cost |
| `dim_products` | Stores product details | product_id (PK), product_category_name, product_weight_g, price |
| `dim_customers` | Stores customer details | customer_id (PK), geolocation_zip_code_prefix (FK), city, state |

## 🔧 ETL Pipeline Implementation

### Extraction
- `chunked_extract_sqlite()`: Batch extraction from Olist SQLite
- `extract_online_sales()`: Process Online Sales CSV in chunks
- `extract_flipkart_products()`: Parse JSON and normalize nested data

### Transformation
- `transform_products()`: Standardize and merge product data, impute missing weights
- `transform_customers()`: Validate and standardize customer data
- `transform_sales()`: Combine Olist and online sales data, calculate prices

### Loading
- `optimized_load()`: Batch loading using `executemany()` for efficiency
- SQLite database: `retail_warehouse.db`

## 📈 OLAP Operations & Results

### Roll-up Analysis
**Top Revenue-Generating Categories:**
1. **beleze_saude (Beauty & Health)**: Total Revenue: $1,258,681.34 | Orders: 8,836
2. **relogios_presentes (Watches & Gifts)**
3. **cama_mesa_banho (Bedding & Bath)**

### Drill-Down Analysis
**Top Products in "cool_stuff" category:**
1. Product ID `5f504b3a1c75b73d6151be81eb05bdc9`: $37,733.90
2. Product ID `165f86fe8b799a708a20ee4ba125c289`: $17,820.91
3. Product ID `601a360bd2a916ecef0e88de72a6531a`: $15,159.81


