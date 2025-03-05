# Inventory Devaluation Analysis

## Table of Contents
-[Project Overview](#project-overview)

-[Data Sources](#data-sources)

### Project Overview
This project analyzes the financial impact of a recent price discount on a bulk of auto parts in the company's catalog. The company has agreements with its customers that allow them to return purchased items, and as a result, customers are requesting compensation due to the devaluation of their inventory. To assess the devaluation, we processed the customer-shared inventory data, which consists of the discounted parts, and used M language to evaluate the financial loss caused by the price reductions.

### Data Sources
1) 2018-2023 Sales Data : "sales_2018-2023.csv" file containing detailed accounting information about each sale made in years 2018-2023 by the company
2) 2024 Sales Data : "sales_2024.csv" file containing detailed accounting information about each sale made in 2024 by the company
3) 2024 Sales Data : "sales_2025.csv" file containing detailed accounting information about each sale made in 2025 by the company
4) Customer Inventory Data : The file "flow.xlsx" contains customer inventory data, including details such as inventory count, previous price, discounted price, and various inventory flow metrics. The dataset includes columns for `Line Code`, `Part Number`, `Short Description`, `In Stock Quantity`, `Old Price`, `New Price`, `Price Difference`, `Percentage Difference`, `Old Inventory Flow`, `New Inventory Flow`, `Inventory Difference`, and `Percentage Difference`.

### Tools
- **Power Query (M Language)**: For data transformation and analysis.
- **Excel / Power BI**: To visualize and report findings.

## Data Cleaning & Preparation

1) Data loading and inspection
```m
SELECT *
FROM table 1
```
   
## Methodology
1. **Data Collection**: The customers provided their inventory data, listing parts that were affected by the price discount. The Company has its own Sales Data.
2. **Data Processing**: Using Power Query M language, the project:
   - Extracts relevant data from the provided inventory.
   - Matches it with the new discounted prices.
   - Computes the devaluation by comparing previous prices with the new ones.
3. **Analysis & Reporting**:
   - The devaluation per part is calculated.
   - Aggregated reports show total financial impact per customer.
   - Visual insights are generated where applicable.



## How to Use
1. Clone this repository.
2. Open the Power Query script in Power BI or Excel.
3. Duplicate the steps for your own needs.

## Expected Outcome
- A clear summary of financial impact per customer.
- A breakdown of inventory devaluation per part.
- A report to assist in decision-making regarding customer compensation claims.

## Contribution & Future Improvements
- Automating data extraction and integration with real-time price updates.
- Expanding the scope to predict future devaluation risks.

Feel free to contribute by submitting pull requests or raising issues for improvements!

---
### Contact
For questions or suggestions, please reach out to [Kuvelet](https://github.com/Kuvelet).
