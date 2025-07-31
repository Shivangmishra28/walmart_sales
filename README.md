# walmart_sales_mysql
## Project Overview

This project is an end-to-end SQL analysis solution designed to extract key business insights from Walmart sales data using **MySQL**. The objective is to perform structured querying, business problem-solving, and data summarization using SQL techniques like aggregation, window functions, filtering, and joins. 

---

## Project Steps

### 1. Set Up the Environment
- **Tools Used**: Visual Studio Code (VS Code), MySQL Workbench, MySQL CLI
- **Goal**: Create a structured SQL-based data analysis solution that can be version-controlled on GitHub.

### 2. Get the Dataset
- **Data Source**: [Walmart Sales Dataset on Kaggle](https://www.kaggle.com/najir0123/walmart-10k-sales-datasets)
- **Storage**: Save the CSV file into the `data/` folder.

### 3. Create the Table in MySQL
- Create a table structure in MySQL to match the dataset:
```sql
CREATE TABLE walmart (
    invoice_id VARCHAR(20),
    branch VARCHAR(10),
    city VARCHAR(50),
    customer_type VARCHAR(50),
    gender VARCHAR(10),
    product_line VARCHAR(100),
    unit_price FLOAT,
    quantity INT,
    tax FLOAT,
    total FLOAT,
    date DATE,
    time TIME,
    payment_method VARCHAR(20),
    cogs FLOAT,
    gross_margin_pct FLOAT,
    gross_income FLOAT,
    rating FLOAT,
    category VARCHAR(50),
    profit_margin FLOAT
);
```
### 4. Install Required Libraries and Load Data
   - **Libraries**: Install necessary Python libraries using:
     ```bash
     pip install pandas numpy sqlalchemy mysql-connector-python psycopg2
     ```
   - **Loading Data**: Read the data into a Pandas DataFrame for initial analysis and transformations.

### 5. Explore the Data
   - **Goal**: Conduct an initial data exploration to understand data distribution, check column names, types, and identify potential issues.
   - **Analysis**: Use functions like `.info()`, `.describe()`, and `.head()` to get a quick overview of the data structure and statistics.

### 6. Data Cleaning
   - **Remove Duplicates**: Identify and remove duplicate entries to avoid skewed results.
   - **Handle Missing Values**: Drop rows or columns with missing values if they are insignificant; fill values where essential.
   - **Fix Data Types**: Ensure all columns have consistent data types (e.g., dates as `datetime`, prices as `float`).
   - **Currency Formatting**: Use `.replace()` to handle and format currency values for analysis.
   - **Validation**: Check for any remaining inconsistencies and verify the cleaned data.

### 7. Feature Engineering
   - **Create New Columns**: Calculate the `Total Amount` for each transaction by multiplying `unit_price` by `quantity` and adding this as a new column.
   - **Enhance Dataset**: Adding this calculated field will streamline further SQL analysis and aggregation tasks.

### 8. Load Data into MySQL and PostgreSQL
   - **Set Up Connections**: Connect to MySQL and PostgreSQL using `sqlalchemy` and load the cleaned data into each database.
   - **Table Creation**: Set up tables in both MySQL and PostgreSQL using Python SQLAlchemy to automate table creation and data insertion.
   - **Verification**: Run initial SQL queries to confirm that the data has been loaded accurately.

### 9. SQL Analysis: Complex Queries and Business Problem Solving
   - **Business Problem-Solving**: Write and execute complex SQL queries to answer critical business questions, such as
-- walmart_sales_project.sql

-- Q1: Find different payment methods and number of transactions, number of qty sold
```
SELECT
  payment_method,
  COUNT(*) AS no_payment,
  SUM(quantity) AS no_qty_sold
FROM walmart
GROUP BY payment_method;
```
-- Q2: Identify the highest rated category in each branch, displaying branch, category, and average rating
```
SELECT * FROM (
  SELECT 
    branch,
    category,
    AVG(rating) AS avg_rating,
    RANK() OVER (PARTITION BY branch ORDER BY AVG(rating) DESC) AS rank_d
  FROM walmart
  GROUP BY branch, category
) AS tb1
WHERE rank_d = 1;

```

-- Q3: Identify the busiest day for each branch based on number of transactions

```
SELECT * FROM (
  SELECT 
    branch,
    DATE_FORMAT(date, '%Y-%m-%d') AS day_name,
    COUNT(*) AS no_transaction,
    RANK() OVER (PARTITION BY branch ORDER BY COUNT(*) DESC) AS rank_d
  FROM walmart
  GROUP BY branch, DATE_FORMAT(date, '%Y-%m-%d')
) AS 
WHERE rank_d = 1;
```

-- Q4: Calculate total quantity of items sold per payment method

```
SELECT 
  payment_method,
  SUM(quantity) AS no_qty_sold
FROM walmart
GROUP BY payment_method;
```

-- Q5: Determine the average, minimum, and maximum rating of categories for each city
SELECT 
```
  city,
  category,
  MIN(rating) AS min_rating,
  MAX(rating) AS max_rating,
  AVG(rating) AS avg_rating
FROM walmart
GROUP BY city, category;
```

-- Q6: Calculate total profit for each category
-- total_profit = unit_price * quantity * profit_margin
```
SELECT
  category,
  SUM(total) AS total_revenue,
  SUM(total * profit_margin) AS profit
FROM walmart
GROUP BY category;
```

-- Q7: Determine the most common payment method for each Branch

```
WITH cte AS (
  SELECT
    branch,
    payment_method,
    COUNT(*) AS total_trans
  FROM walmart
  GROUP BY branch, payment_method
),
ranked AS (
  SELECT *,
    RANK() OVER (PARTITION BY branch ORDER BY total_trans DESC) AS rank1
  FROM cte
)
SELECT *
FROM ranked
WHERE rank1 = 1;
```

-- Q8: Categorize sales by shift (Morning, Afternoon, Evening) and find number of invoices per shift
```
SELECT 
  branch,
  CASE
    WHEN HOUR(TIME(time)) < 12 THEN 'Morning'
    WHEN HOUR(TIME(time)) BETWEEN 12 AND 17 THEN 'Afternoon'
    ELSE 'Evening'
  END AS day_time,
  COUNT(*) AS invoice_count
FROM walmart
GROUP BY branch, day_time;
```

-- Q9: Identify top 5 branches with highest decrease in revenue from 2022 to 2023
-- (last_rev - curr_rev)/last_rev * 100
```
WITH revenue_2022 AS (
  SELECT branch, SUM(total) AS rev_2022
  FROM walmart
  WHERE YEAR(STR_TO_DATE(date, '%d/%m/%y')) = 2022
  GROUP BY branch
),
revenue_2023 AS (
  SELECT branch, SUM(total) AS rev_2023
  FROM walmart
  WHERE YEAR(STR_TO_DATE(date, '%d/%m/%y')) = 2023
  GROUP BY branch
),
combined AS (
  SELECT 
    r22.branch,
    r22.rev_2022,
    r23.rev_2023,
    ((r22.rev_2022 - r23.rev_2023) / r22.rev_2022) * 100 AS drop_ratio
  FROM revenue_2022 r22
  JOIN revenue_2023 r23 ON r22.branch = r23.branch
)
SELECT * FROM combined
ORDER BY drop_ratio DESC
LIMIT 5;
```


   - **Documentation**: Keep clear notes of each query's objective, approach, and results.

### 10. Project Publishing and Documentation
   - **Documentation**: Maintain well-structured documentation of the entire process in Markdown or a Jupyter Notebook.
   - **Project Publishing**: Publish the completed project on GitHub or any other version control platform, including:
     - The `README.md` file (this document).
     - Jupyter Notebooks (if applicable).
     - SQL query scripts.
     - Data files (if possible) or steps to access them.

---

## Requirements

- **Python 3.8+**
- **SQL Databases**: MySQL, PostgreSQL
- **Python Libraries**:
  - `pandas`, `numpy`, `sqlalchemy`, `mysql-connector-python`, `psycopg2`
- **Kaggle API Key** (for data downloading)

## Getting Started

1. Clone the repository:
   ```bash
   git clone <repo-url>
   ```
2. Install Python libraries:
   ```bash
   pip install -r requirements.txt
   ```
3. Set up your Kaggle API, download the data, and follow the steps to load and analyze.

---

## Project Structure

```plaintext
|-- data/                     # Raw data and transformed data
|-- sql_queries/              # SQL scripts for analysis and queries
|-- notebooks/                # Jupyter notebooks for Python analysis
|-- README.md                 # Project documentation
|-- requirements.txt          # List of required Python libraries
|-- main.py                   # Main script for loading, cleaning, and processing data
```
---

## Results and Insights

This section will include your analysis findings:
- **Sales Insights**: Key categories, branches with highest sales, and preferred payment methods.
- **Profitability**: Insights into the most profitable product categories and locations.
- **Customer Behavior**: Trends in ratings, payment preferences, and peak shopping hours.

## Future Enhancements

Possible extensions to this project:
- Integration with a dashboard tool (e.g., Power BI or Tableau) for interactive visualization.
- Additional data sources to enhance analysis depth.
- Automation of the data pipeline for real-time data ingestion and analysis.

---

## License

This project is licensed under the MIT License. 

---

## Acknowledgments

- **Data Source**: Kaggle’s Walmart Sales Dataset
- **Inspiration**: Walmart’s business case studies on sales and supply chain optimization.

---


