# Inventory Devaluation Analysis

### Table of Contents
---
-[Project Overview](#project-overview)

-[Data Sources](#data-sources)

-[Data Cleaning & Preparation](#data-cleaning--preparation)

-[Analysis](#analysis)

-[Results & Findings](#results--findings)

-[Next Steps](#next-steps)

### Project Overview
---
This project evaluates the financial impact of a recent price reduction on a bulk of auto parts in the company's catalog. Due to agreements that allow customers to return purchased items, many have requested compensation for the devaluation of their inventory. To assess the extent of this impact, I processed customer-shared inventory data, focusing on discounted parts, and utilized M language to quantify the financial loss. Additionally, I compared customer inventory devaluation with our own sales data to gain a comprehensive understanding of the overall effect of the price reductions.

### Data Sources
---
**1.** 2018-2023 Sales Data : "sales_2018-2023.csv" file containing detailed accounting information about each sale made in years 2018-2023 by the company. The dataset includes columns for `Customer ID`, `Customer Name`, `Invoice/CM #`, `Apply to Invoice Number`, `Credit Memo`, `Progress Billing Invoice`, `Date`, `Ship By`, `Quote`, `Quote #`, `Quote Good Thru Date`, `Drop Ship`, `Ship to Name`, `Ship to Address-Line One`, `Ship to Address-Line Two`, `Ship to City`, `Ship to State`, `Ship to Zipcode`, `Ship to Country`, `Customer PO`, `Ship Via`, `Ship Date`, `Date Due`, `Discount Amount`, `Discount Date`, `Displayed Terms`, `Sales Representative ID`, `Accounts Receivable Account`, `Accounts Receivable Amount`, `Sales Tax ID`, `Invoice Note`, `Note Prints After Line Items`, `Statement Note`, `Stmt Note Prints Before Ref`, `Internal Note`, `Beginning Balance Transaction`, `AR Date Cleared in Bank Rec`, `Number of Distributions`, `Invoice/CM Distribution`, `Apply to Invoice Distribution`, `Apply To Sales Order`, `Apply to Proposal`, `Item ID`, `Quantity`, `SO/Proposal Number`, `Serial Number`, `SO/Proposal Distribution`, `G/L Account`, `GL Date Cleared in Bank Rec`, `Unit Price`, `Tax Type`, `UPC / SKU`, `Weight`, `Amount`, `Inventory Account`, `Inv Acnt Date Cleared In Bank Rec`, `Cost of Sales Account`, `COS Acnt Date Cleared In Bank Rec`, `U/M ID`, `U/M No. of Stocking Units`, `Stocking Quantity`, `Stocking Unit Price`, `Cost of Sales Amount`, `Job ID`, `Sales Tax Agency ID`, `Transaction Period`, `Transaction Number`, `Receipt Number`, `Return Authorization`, `Voided by Transaction`, `Retainage Percent`, `Recur Number`, `Recur Frequency`, `Description`

**2.** 2024 Sales Data : "sales_2024.csv" file containing detailed accounting information about each sale made in 2024 by the company

**3.** 2024 Sales Data : "sales_2025.csv" file containing detailed accounting information about each sale made in 2025 by the company

**4.** Customer Inventory Data : The file "flow.xlsx" contains customer inventory data, including details such as inventory count, previous price, discounted price, and various inventory flow metrics. The dataset includes columns for `Line Code`, `Part Number`, `Short Description`, `In Stock Quantity`, `Old Price`, `New Price`, `Price Difference`, `Percentage Difference`, `Old Inventory Flow`, `New Inventory Flow`, `Inventory Difference`, and `Percentage Difference`.

### Tools
---
- **Power Query (M Language)**: For data transformation and analysis.
- **Excel / Power BI**: To visualize and report findings.

### Data Cleaning & Preparation
---
**1.** This Power Query (M) script combines multiple sales tables, removes unnecessary columns, renames and transforms key fields (including converting amounts to negative values), ensures correct data types, and replaces blank values with "(blank)" for better data consistency and readability.

```powerquery
let
    // Combine multiple sales tables into one
    Source = Table.Combine({SALES_SUS_2018_2023, SALES_SUS_2024, SALES_SUS_2025}),
    
    // Remove unnecessary columns
    RemovedColumns = Table.RemoveColumns(
        Source, 
        {
            "Apply to Invoice Number", "Progress Billing Invoice", "Ship By", "Quote", "Quote #", 
            "Quote Good Thru Date", "Ship Via", "Ship Date", "Date Due", "Sales Tax ID", 
            "Invoice Note", "Note Prints After Line Items", "Statement Note", "Stmt Note Prints Before Ref", 
            "Internal Note", "Beginning Balance Transaction", "AR Date Cleared in Bank Rec", 
            "Number of Distributions", "Invoice/CM Distribution", "Apply to Invoice Distribution", 
            "Apply To Sales Order", "Apply to Proposal", "Serial Number", "SO/Proposal Distribution", 
            "Weight", "Stocking Quantity", "Stocking Unit Price", "Return Authorization", 
            "Receipt Number", "Voided by Transaction", "Retainage Percent", "Recur Number", 
            "Recur Frequency", "Ship to Name", "Ship to Address-Line One", "Ship to Address-Line Two", 
            "Discount Amount", "Discount Date", "Displayed Terms", "Accounts Receivable Account", 
            "Accounts Receivable Amount", "SO/Proposal Number", "GL Date Cleared in Bank Rec", 
            "Tax Type", "UPC / SKU", "Inv Acnt Date Cleared In Bank Rec", "COS Acnt Date Cleared In Bank Rec", 
            "U/M ID", "U/M No. of Stocking Units", "Job ID", "Sales Tax Agency ID", 
            "Transaction Period", "Transaction Number", "Description", "Inventory Account"
        }
    ),

    // Rename "Item ID" to "Sold AS (Item ID)"
    RenamedItemID = Table.RenameColumns(RemovedColumns, {{"Item ID", "Sold AS (Item ID)"}}),

    // Multiply "Amount" by -1
    AdjustedAmount = Table.TransformColumns(RenamedItemID, {{"Amount", each _ * -1, Currency.Type}}),

    // Rename relevant columns for clarity
    RenamedColumns = Table.RenameColumns(
        AdjustedAmount, 
        {
            {"Amount", "Sales_Amount"}, 
            {"Quantity", "Sales_Quantity"}, 
            {"Unit Price", "Sales_Unit Price"}
        }
    ),

    // Change column type for "Cost of Sales Amount"
    ChangedType = Table.TransformColumnTypes(RenamedColumns, {{"Cost of Sales Amount", type number}}),

    // Replace blank values with "(blank)" in "Sold AS (Item ID)"
    ReplacedBlanks = Table.ReplaceValue(
        ChangedType, 
        "", 
        "(blank)", 
        Replacer.ReplaceValue, 
        {"Sold AS (Item ID)"}
    )

in
    ReplacedBlanks

```

**2.** This Power Query (M) script loads an Excel file, extracts data from the "Flow" sheet, promotes headers, assigns appropriate data types, and creates a new column by concatenating the "Part #" and "In Stock" values. The concatenated column will later be used to merge two queries for further analysis.

```powerquery
let
    // Load the Excel workbook and extract the "Flow" sheet
    Source = Excel.Workbook(
        File.Contents("C:\Users\ckuvelet\OneDrive - ISC Industries\Desktop\DeVal\Flow.xlsx"), 
        null, 
        true
    ),
    
    // Access the sheet named "Flow"
    Flow_Sheet = Source{[Item="Flow", Kind="Sheet"]}[Data],

    // Promote the first row to headers
    PromotedHeaders = Table.PromoteHeaders(Flow_Sheet, [PromoteAllScalars=true]),

    // Change column data types for consistency
    ChangedType = Table.TransformColumnTypes(
        PromotedHeaders, 
        {
            {"Line Code", type text}, 
            {"Part #", type text}, 
            {"In Stock", Int64.Type}, 
            {"Short Description", type text}, 
            {"Old Price", type number}, 
            {"New Price", type number}, 
            {"Difference", type number}, 
            {"% Difference", type number}, 
            {"Old Inventory Flow", type number}, 
            {"New Inventory Flow", type number}, 
            {"Difference_1", type number}, 
            {"% Difference_2", type number}
        }
    ),

    // Add a new column that concatenates "Part #" and "In Stock" values
    AddedCustom = Table.AddColumn(
        ChangedType, 
        "Concatenated_Part&Stock", 
        each [#"Part #"] & "_" & Text.From([In Stock])
    )

in
    AddedCustom
```

### Analysis
---
**1.** This Power Query (M) script joins the "Flow" table with "Sales_SUS," filters relevant sales data, and processes it by sorting, indexing, and calculating cumulative sales quantity and amount for each part. It also replaces rows by expanding a repeated list for each sales quantity to create individual transaction rows. Finally, it expands the grouped data back into a structured format and creates a unique reference column for further analysis.

```powerquery
let
    // Step 1: Join and Clean Data
    // Merges the "Flow" table with "Sales_SUS" on "Part #" and "Sold AS (Item ID)" using a Left Outer Join.
    Source = Table.NestedJoin(Flow, {"Part #"}, Sales_SUS, {"Sold AS (Item ID)"}, "Sales_SUS", JoinKind.LeftOuter),

    // Removes unnecessary columns to keep only relevant data for further analysis.
    RemovedColumns = Table.RemoveColumns(Source, 
        {"Line Code", "Short Description", "Old Price", "New Price", "Difference", "% Difference", 
        "Old Inventory Flow", "New Inventory Flow", "In Stock", "Difference_1", "% Difference_2", "Concatenated_Part&Stock"}),

    // Expands the "Sales_SUS" table and renames columns for clarity.
    ExpandedSales_SUS = Table.ExpandTableColumn(RemovedColumns, "Sales_SUS", 
        {"Customer ID", "Invoice/CM #", "Credit Memo", "Date", "Customer PO", "Sales_Quantity", "Sales_Unit Price"}, 
        {"S_.Customer ID", "S_.Invoice/CM #", "S_.Credit Memo", "S_.Date", "S_.Customer PO", "S_.Sales_Quantity", "S_.Sales_Unit Price"}),

    // Step 2: Filter Data
    // Filters records where "Customer ID" is "USPAAU".
    USPAUUFiltered = Table.SelectRows(ExpandedSales_SUS, each ([S_.Customer ID] = "USPAAU")),

    // Further filters records to keep only those with positive sales quantity.
    SalesQtyFiltered = Table.SelectRows(USPAUUFiltered, each [S_.Sales_Quantity] > 0),

    // Removes transactions that are marked as "Credit Memo" (i.e., refunds or credit notes).
    CreditMemosFiltered = Table.SelectRows(SalesQtyFiltered, each ([S_.Credit Memo] = false)),

    // Step 3: Add Index and Calculate Sales Columns
    // Adds an index column to keep track of the original order.
    AddedOriginalIndex = Table.AddIndexColumn(CreditMemosFiltered, "Original_Index", 1, 1, Int64.Type),

    // Creates a repeated list of 1s for each sales quantity.
    AddedSalesQtyList = Table.AddColumn(AddedOriginalIndex, "Sales_Qty_(1)", each List.Repeat({1}, [S_.Sales_Quantity])),

    // Expands the repeated list into separate rows.
    ExpandedSalesQtyList = Table.ExpandListColumn(AddedSalesQtyList, "Sales_Qty_(1)"),

    // Creates a new column that calculates individual sales amounts.
    SalesAmount_1 = Table.AddColumn(ExpandedSalesQtyList, "Sales_Amount_(1)", each [#"Sales_Qty_(1)"] * [S_.Sales_Unit Price]),

    // Step 4: Sort Data
    // Sorts by "Part #" in ascending order and "Date" in descending order to organize transactions.
    SortedData = Table.Sort(SalesAmount_1, {{"Part #", Order.Ascending}, {"S_.Date", Order.Descending}}),

    // Step 5: Add an Index Column for Cumulative Calculation
    AddedAllIndex = Table.AddIndexColumn(SortedData, "Index", 1, 1, Int64.Type),

    // Step 6: Group Data by "Part #"
    // Groups all transactions for each unique "Part #".
    GroupedRows = Table.Group(AddedAllIndex, {"Part #"}, 
        {{"Grouped Data", each _, 
            type table [#"Part #"=nullable text, S_.Customer ID=nullable text, #"S_.Invoice/CM #"=nullable text, 
            S_.Credit Memo=nullable logical, S_.Date=nullable date, S_.Customer PO=nullable text, 
            S_.Sales_Quantity=nullable number, S_.Sales_Unit Price=nullable number, Original_Index=number, 
            #"Sales_Qty_(1)"=number, #"Sales_Amount_(1)"=number, Index=number]}
        }),

    // Step 7: Calculate Cumulative Sales Quantity
    ModifiedGroupedRows_Qty = Table.TransformColumns(GroupedRows, 
        {{"Grouped Data", each Table.AddColumn(_, "Cumulative Sales_Qty_(1)", 
            (currentRow) =>
                List.Sum(List.FirstN([#"Sales_Qty_(1)"], List.PositionOf([Index], currentRow[Index]) + 1))
        ), 
        type table [#"Part #"=nullable text, S_.Customer ID=nullable text, #"S_.Invoice/CM #"=nullable text, 
        S_.Credit Memo=nullable logical, S_.Date=nullable date, S_.Customer PO=nullable text, S_.Sales_Quantity=nullable number, 
        S_.Sales_Unit Price=nullable number, Original_Index=number, #"Sales_Qty_(1)"=number, 
        #"Sales_Amount_(1)"=number, Index=number, #"Cumulative Sales_Qty_(1)"=number]}
    }),

    // Step 8: Calculate Cumulative Sales Amount
    ModifiedGroupedRows_Amount = Table.TransformColumns(ModifiedGroupedRows_Qty, 
        {{"Grouped Data", each Table.AddColumn(_, "Cumulative Sales_Amount_(1)", 
            (currentRow) =>
                List.Sum(List.FirstN([#"Sales_Amount_(1)"], List.PositionOf([Index], currentRow[Index]) + 1))
        ), 
        type table [#"Part #"=nullable text, S_.Customer ID=nullable text, #"S_.Invoice/CM #"=nullable text, 
        S_.Credit Memo=nullable logical, S_.Date=nullable date, S_.Customer PO=nullable text, S_.Sales_Quantity=nullable number, 
        S_.Sales_Unit Price=nullable number, Original_Index=number, #"Sales_Qty_(1)"=number, #"Sales_Amount_(1)"=number, 
        Index=number, #"Cumulative Sales_Qty_(1)"=number, #"Cumulative Sales_Amount_(1)"=number]}
    }),

    // Step 9: Expand Grouped Data
    // Extracts all the grouped columns back into a flat table format.
    ExpandedGroupedData = Table.ExpandTableColumn(ModifiedGroupedRows_Amount, "Grouped Data", 
        {"S_.Customer ID", "S_.Invoice/CM #", "S_.Credit Memo", "S_.Date", "S_.Customer PO", 
        "S_.Sales_Quantity", "S_.Sales_Unit Price", "Original_Index", "Sales_Qty_(1)", 
        "Sales_Amount_(1)", "Index", "Cumulative Sales_Qty_(1)", "Cumulative Sales_Amount_(1)"}),

    // Step 10: Create a Unique Concatenated Column for Reference
    // Adds a new column that combines "Part #" and "Cumulative Sales Quantity" for easier tracking.
    ConcatenatedPartCumSum = Table.AddColumn(ExpandedGroupedData, "Concatenated_Part&CumSum", 
        each [#"Part #"] & "_" & Text.From([#"Cumulative Sales_Qty_(1)"]))

in
    ConcatenatedPartCumSum
```
**2.** This Power Query (M) script performs a Left Outer Join between Flow and Flow_Deval_Project using "Concatenated_Part&Stock" from Flow and "Concatenated_Part&CumSum" from Flow_Deval_Project, extracts relevant sales data, renames key cumulative sales columns for clarity, and removes unnecessary fields to create a clean dataset

```powerquery
let
    // Step 1: Perform a Left Outer Join between "Flow" and "Flow_Deval_Project"
    // - Merging on "Concatenated_Part&Stock" from "Flow" and "Concatenated_Part&CumSum" from "Flow_Deval_Project"
    Source = Table.NestedJoin(
        Flow, 
        {"Concatenated_Part&Stock"}, 
        Flow_Deval_Project, 
        {"Concatenated_Part&CumSum"}, 
        "Flow_Deval_Project", 
        JoinKind.LeftOuter
    ),

    // Step 2: Expand the "Flow_Deval_Project" table to extract relevant columns
    ExpandedFlowDevalProject = Table.ExpandTableColumn(
        Source, 
        "Flow_Deval_Project", 
        {
            "Part #", "S_.Customer ID", "S_.Invoice/CM #", "S_.Credit Memo", "S_.Date", "S_.Customer PO",
            "S_.Sales_Quantity", "S_.Sales_Unit Price", "Original_Index", "Sales_Qty_(1)", "Sales_Amount_(1)",
            "Index", "Cumulative Sales_Qty_(1)", "Cumulative Sales_Amount_(1)", "Concatenated_Part&CumSum"
        },
        {
            "Flow_Deval_Project.Part #", "Flow_Deval_Project.S_.Customer ID", "Flow_Deval_Project.S_.Invoice/CM #",
            "Flow_Deval_Project.S_.Credit Memo", "Flow_Deval_Project.S_.Date", "Flow_Deval_Project.S_.Customer PO",
            "Flow_Deval_Project.S_.Sales_Quantity", "Flow_Deval_Project.S_.Sales_Unit Price",
            "Flow_Deval_Project.Original_Index", "Flow_Deval_Project.Sales_Qty_(1)", "Flow_Deval_Project.Sales_Amount_(1)",
            "Flow_Deval_Project.Index", "Flow_Deval_Project.Cumulative Sales_Qty_(1)", 
            "Flow_Deval_Project.Cumulative Sales_Amount_(1)", "Flow_Deval_Project.Concatenated_Part&CumSum"
        }
    ),

    // Step 3: Rename selected columns for better readability
    RenamedColumns = Table.RenameColumns(
        ExpandedFlowDevalProject,
        {
            {"Flow_Deval_Project.Cumulative Sales_Amount_(1)", "USPAAU_Cumulative_Sales_Amount"},
            {"Flow_Deval_Project.Cumulative Sales_Qty_(1)", "USPAAU_Cumulative_Sales_Qty"}
        }
    ),

    // Step 4: Remove unnecessary columns to clean up the dataset
    RemovedColumns = Table.RemoveColumns(
        RenamedColumns, 
        {
            "Line Code", "Short Description", "Old Price", "New Price", "Difference", "% Difference", 
            "Old Inventory Flow", "New Inventory Flow", "Difference_1", "% Difference_2", "Concatenated_Part&Stock",
            "Flow_Deval_Project.Part #", "Flow_Deval_Project.S_.Customer ID", "Flow_Deval_Project.S_.Invoice/CM #", 
            "Flow_Deval_Project.S_.Credit Memo", "Flow_Deval_Project.S_.Date", "Flow_Deval_Project.S_.Customer PO", 
            "Flow_Deval_Project.S_.Sales_Quantity", "Flow_Deval_Project.S_.Sales_Unit Price", 
            "Flow_Deval_Project.Original_Index", "Flow_Deval_Project.Sales_Qty_(1)", 
            "Flow_Deval_Project.Sales_Amount_(1)", "Flow_Deval_Project.Index", 
            "Flow_Deval_Project.Concatenated_Part&CumSum"
        }
    )

in
    RemovedColumns
```

### Results & Findings
---
After processing and analyzing the customer inventory data and sales records, we identified the cumulative sales value for each SKU, sorted by the most recent date for the target customer. The findings highlight the financial impact of price reductions and the extent of inventory devaluation.

#### **Key Findings:**
- The cumulative sales quantity and amount for each SKU were calculated, ensuring accurate financial assessment.
- The analysis considered only the SKUs impacted by the price reductions and matched them with customer inventory data.
- By filtering the dataset based on inventory count and the requested SKU numbers, we ensured a focused approach to understanding compensation claims.
- The final dataset was structured and optimized for further reporting and visualization in Power BI.

#### **Next Steps:**
1. **Upload to Power BI:**  
   - The latest query results, filtered and structured, will be uploaded to the Power BI company server for further analysis.
   
2. **Access in Excel:**  
   - To retrieve the queried table from Power BI Services in Excel:  
     - Navigate to the **Insert** tab.  
     - Click on **PivotTable** and select **From Power BI**.  
     - Choose the relevant dataset from Power BI and insert the table for further analysis.

3. **Further Analysis & Reporting:**  
   - Utilize Power BI dashboards and Excel reports to visualize the financial impact of inventory devaluation.
   - Share insights with stakeholders to support decision-making regarding compensation and pricing strategies.


### Contact
For questions or suggestions, please reach out to [Kuvelet](https://github.com/Kuvelet).
