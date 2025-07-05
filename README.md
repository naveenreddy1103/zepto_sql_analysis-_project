# 🛒 Zepto E-commerce SQL Data Analyst Portfolio Project
This is a complete, real-world data analyst portfolio project based on an e-commerce inventory dataset scraped from [Zepto](https://www.zeptonow.com/) — one of India’s fastest-growing quick-commerce startups. This project simulates real analyst workflows, from raw data exploration to business-focused data analysis.


## 📌 Project Overview

The goal is to simulate how actual data analysts in the e-commerce or retail industries work behind the scenes to use SQL to:

✅ Set up a messy, real-world e-commerce inventory **database**

✅ Perform **Exploratory Data Analysis (EDA)** to explore product categories, availability, and pricing inconsistencies

✅ Implement **Data Cleaning** to handle null values, remove invalid entries, and convert pricing from paise to rupees

✅ Write **business-driven SQL queries** to derive insights around **pricing, inventory, stock availability, revenue** and more

## 📁 Dataset Overview
The dataset was sourced from [Kaggle](https://www.kaggle.com/datasets/palvinder2006/zepto-inventory-dataset/data?select=zepto_v2.csv) and was originally scraped from Zepto’s official product listings. It mimics what you’d typically encounter in a real-world e-commerce inventory system.

Each row represents a unique SKU (Stock Keeping Unit) for a product. Duplicate product names exist because the same product may appear multiple times in different package sizes, weights, discounts, or categories to improve visibility – exactly how real catalog data looks.

🧾 Columns:
- **sku_id:** Unique identifier for each product entry (Synthetic Primary Key)

- **name:** Product name as it appears on the app

- **category:** Product category like Fruits, Snacks, Beverages, etc.

- **mrp:** Maximum Retail Price (originally in paise, converted to ₹)

- **discountPercent:** Discount applied on MRP

- **discountedSellingPrice:** Final price after discount (also converted to ₹)

- **availableQuantity:** Units available in inventory

- **weightInGms:** Product weight in grams

- **outOfStock:** Boolean flag indicating stock availability

- **quantity:** Number of units per package (mixed with grams for loose produce)

## 🔧 Project Workflow

Here’s a step-by-step breakdown of what we do in this project:

1.  Database & Table Creation
We start by creating a SQL Server table with appropriate data types:
CREATE TABLE zepto (
  sku_id INT IDENTITY(1,1) PRIMARY KEY,
  category VARCHAR(120),
  name VARCHAR(150) NOT NULL,
  mrp DECIMAL(8,2),
  discountPercent DECIMAL(5,2),
  availableQuantity INT,
  discountedSellingPrice DECIMAL(8,2),
  weightInGms INT,
  outOfStock BIT,
  quantity INT
);

2.  Data Import
   -Loaded CSV using SQL Server Management Studio’s Import Flat File Wizard
(Right-click on the database → Tasks → Import Flat File)

  - If you're not able to use the GUI import feature, use this code instead:
BULK INSERT zepto
FROM 'C:\path\to\your\zepto_v2.csv'
WITH (
    FIRSTROW = 2,
    FIELDTERMINATOR = ',',
    ROWTERMINATOR = '\n',
    TEXTQUALIFIER = '"',
    CODEPAGE = '65001', -- UTF-8 Encoding
    TABLOCK
);

   - Faced encoding issues (UTF-8 error), which were fixed by saving the CSV file as CSV UTF-8 (Comma delimited) using Excel or Notepad.

3.  Data Exploration
-Total number of records:
SELECT COUNT(*) AS TotalRecords FROM zepto;

-Sample data:
SELECT TOP 10 * FROM zepto;

-Null values per column:
SELECT 
  SUM(CASE WHEN category IS NULL THEN 1 ELSE 0 END) AS Null_Category,
  SUM(CASE WHEN name IS NULL THEN 1 ELSE 0 END) AS Null_Name,
  SUM(CASE WHEN mrp IS NULL THEN 1 ELSE 0 END) AS Null_MRP,
  SUM(CASE WHEN discountPercent IS NULL THEN 1 ELSE 0 END) AS Null_Discount,
  SUM(CASE WHEN discountedSellingPrice IS NULL THEN 1 ELSE 0 END) AS Null_DSP,
  SUM(CASE WHEN quantity IS NULL THEN 1 ELSE 0 END) AS Null_Quantity
FROM zepto;

-Distinct product categories:
SELECT DISTINCT category FROM zepto;

-In-stock vs out-of-stock count:
SELECT 
  outOfStock, COUNT(*) AS Count
FROM zepto
GROUP BY outOfStock;

-Duplicate product names (multiple SKUs):
SELECT name, COUNT(*) AS Count
FROM zepto
GROUP BY name
HAVING COUNT(*) > 1;


4.  Data Cleaning
-Remove rows with MRP or Discounted Price = 0:
DELETE FROM zepto
WHERE mrp = 0 OR discountedSellingPrice = 0;

-Convert from paise to rupees (if applicable):
UPDATE zepto
SET 
  mrp = mrp / 100.0,
  discountedSellingPrice = discountedSellingPrice / 100.0
WHERE mrp > 1000;  -- Optional: use a filter if already in ₹

5.  Business Insights
-Top 10 products with highest discounts:
SELECT TOP 10 name, mrp, discountedSellingPrice, discountPercent
FROM zepto
ORDER BY discountPercent DESC;

-Out-of-stock high-MRP products:
SELECT name, mrp
FROM zepto
WHERE outOfStock = 1 AND mrp > 500;

-Estimated potential revenue by category:
SELECT category, SUM(discountedSellingPrice * quantity) AS PotentialRevenue
FROM zepto
GROUP BY category;

-Expensive products with minimal discount:
SELECT name, mrp, discountPercent
FROM zepto
WHERE mrp > 500 AND discountPercent < 5;

-Top 5 categories with highest average discount:
SELECT TOP 5 category, AVG(discountPercent) AS AvgDiscount
FROM zepto
GROUP BY category
ORDER BY AvgDiscount DESC;

-Price per gram (value for money):
SELECT name, discountedSellingPrice, weightInGms,
       (discountedSellingPrice * 1.0 / weightInGms) AS PricePerGram
FROM zepto
WHERE weightInGms > 0
ORDER BY PricePerGram ASC;

-Weight classification (Low < 500g, Medium 500–2000g, Bulk > 2000g):
SELECT *,
  CASE 
    WHEN weightInGms < 500 THEN 'Low'
    WHEN weightInGms BETWEEN 500 AND 2000 THEN 'Medium'
    ELSE 'Bulk'
  END AS WeightCategory
FROM zepto;

-Total inventory weight per category:
SELECT category, SUM(weightInGms * quantity) AS TotalWeight
FROM zepto
GROUP BY category;






## 🛠️ How to Use This Project

1. **Clone the repository**
   ```bash
   git clone https://github.com/amlanmohanty/zepto-SQL-data-analysis-project.git
   cd zepto-SQL-data-analysis-project
   ```
2. **Open zepto_SQL_data_analysis.sql**

    This file contains:

      - Table creation

      - Data exploration

      - Data cleaning

      - SQL Business analysis
  
3. **Load the dataset into pgAdmin or any other PostgreSQL client**

      - Create a database and run the SQL file

      - Import the dataset (convert to UTF-8 if necessary)

