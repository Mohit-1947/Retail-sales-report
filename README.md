# Retail Store Sales - Data Cleaning and Power BI Dashboard

## Overview
This project uses the "Retail Store Sales: Dirty for Data Cleaning" dataset from Kaggle, containing 12,575 rows of synthetic retail transactions across eight product categories, with 25 items per category at static prices. The dataset was intentionally corrupted with missing value "UNKNOWN" entries, and blank fields to simulate real-world data quality issues. The objective was to clean the dataset using Excel and Power Query, verify data integrity using SQL-style logic, and build an interactive Power BI dashboard.

## Tools Used
Excel, Power Query, Power BI, DAX

## Data Cleaning Process

### 1. Handling blank Discount Applied values
The Discount Applied column had blank cells with no way to logically infer a true or false value from other columns. These were filled in Excel rather than left blank, since a blank boolean field causes inconsistent grouping in visuals.

### 2. Recovering missing Price Per Unit
Since each item in this dataset has a static price, blank Price Per Unit values were recovered using an IF-based formula that referenced the item code to look up its known price. This worked reliably because price is fixed per item, unlike quantity or total spent, which vary by transaction.

### 3. Recovering missing Item values
Item was the hardest column to recover because it could not be reversed using a single formula. A lookup table was built separately using Category, Item, and Price Per Unit for all rows where Item was already known. Duplicates were removed from this lookup table so each Category-Price combination pointed to exactly one Item. An INDEX and MATCH formula was then used on the main dataset: it multiplied two logical arrays, one checking if Category matched and one checking if Price matched, to identify the correct row position in the lookup table, then returned the corresponding Item. Cases where no unique match existed were flagged rather than guessed.

### 4. Recovering missing Quantity values
Quantity was recovered using the relationship Total Spent equals Quantity multiplied by Price Per Unit. Where Quantity was blank but both Price Per Unit and Total Spent were valid numbers, Quantity was calculated as Total Spent divided by Price Per Unit. Where recovery was not possible because Total Spent or Price Per Unit was also missing, the value was marked as unknown instead of defaulting to zero, since zero would incorrectly imply no items were purchased.

### 5. Removing duplicate Transaction IDs
Duplicate Transaction IDs were identified in Power Query Editor by grouping on Transaction ID and checking for counts greater than one. Rows confirmed as duplicates were removed to ensure each transaction was counted only once in the final revenue and transaction totals.

### 6. Fixing a data type error in Power BI
After loading the cleaned data into Power BI, a measure summing Total Spent returned an error stating the data reader failed to move to the next row. Investigation in Power Query traced this to a row where Quantity and Total Spent still contained the literal text unknown, which conflicted with the column being typed as a number. This was fixed by replacing the text unknown with a proper null value before the column type conversion step, rather than after, which allowed Power BI to correctly type the column as Decimal Number and resolved the error.

## Key DAX Measures
- Total Revenue: sum of Total Spent
- Total Transactions: distinct count of Transaction ID
- Total Items Sold: sum of Quantity
- Average Order Value: Total Revenue divided by Total Items Sold, using DIVIDE to safely handle any zero denominators

## Dashboard Features
- KPI cards for Total Revenue, Total Transactions, Total Items Sold, and Average Order Value
- Items sold broken down by Category
- Revenue broken down by Category and Discount Applied status
- Average order value trend across categories
- Transactions split by Location, Online versus In-store



