# Inventory Devaluation Analysis

### Table of Contents
---
-[Project Overview](#project-overview)

-[Data Sources](#data-sources)

### Project Overview
---
This project analyzes the financial impact of a recent price discount on a bulk of auto parts in the company's catalog. The company has agreements with its customers that allow them to return purchased items, and as a result, customers are requesting compensation due to the devaluation of their inventory. To assess the devaluation, we processed the customer-shared inventory data, which consists of the discounted parts, and used M language to evaluate the financial loss caused by the price reductions.

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
