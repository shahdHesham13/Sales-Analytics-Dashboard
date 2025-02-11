# Sales Performance Analytics

This project offers a detailed analysis of sales performance through an interactive dashboard helping stakeholders make informed business decisions. The Python script where I explored the dataset a bit and generated key insights.

## Data Sources and Tables

The dataset consists of the first three primary tables:

### 1. **Fact_Sales**
- Contains transactional sales data.
- Key columns: `Date_Time`, `Sales_USD`, `COGs`, `Quantity`

### 2. **Dim_Product**
- Contains product-related information.
- Key column: `Product_ID`

### 3. **Dim_Accounts**
- Contains customer account details.
- Key column: `Account_ID`

### 4. **Dim_Date** (Generated using DAX)
- Created to support time intelligence functions.
- Includes a date hierarchy and an additional column `Inpast` to filter past months only.

```DAX
Dim_Date = CALENDAR(
    DATE(2022, 1, 1),
    DATE(2024, 12, 31)
)
```

#### **Inpast Column**
Used to highlight only past months, excluding future months in reports so that we won't have the empty months.

```DAX 
Inpast =
VAR lastsalesdate = MAX(Fact_Sales[Date_Time])
VAR lastsalesdatePY = EDATE(lastsalesdate,-12)
RETURN
Dim_Date[Date] <= lastsalesdatePY
```

### 5. **SLC_Values** (Slicer Table)

Contains predefined values (`Sales`, `Gross Profit`, `Quantity`) to enable dynamic selection of key metrics across the reports.

## Measures

### **Base Measures**
These foundational calculations provide core metrics for analysis:

- **Gross Profit** = `[Sales] - [COGs]`
- **Quantity** = `SUM(Fact_Sales[Quantity])`
- **Sales** = `SUM(Fact_Sales[Sales_USD])`

### **Dynamic Titles**
Ensures report consistency and enhances usability:

- `_Column Chart Title = SELECTEDVALUE(Slc_Values[Values]) & " YTD & PYTD | Month"`
- `_Report Title = "Plant Co. " & SELECTEDVALUE(Slc_Values[Values]) & " Performance " & SELECTEDVALUE(Dim_Date[Date].[Year])`
- `_Scatter Chart Title = "Account Profitability Segmentation | GP% and " & SELECTEDVALUE(Slc_Values[Values])`
- `_Waterfall Chart Title = SELECTEDVALUE(Slc_Values[Values]) & " YTD vs PYTD | Month - Country - Product"`

### **Performance Metrics**
- **GP% (Gross Profit Percentage)** = `DIVIDE([Gross Profit], [Sales])`

### **Time-Based Measures**

##### **Prior Year to Date (PYTD)**
Used to compare the same period in the previous year:

```DAX
PYTD_GrossProfit = CALCULATE([Gross Profit], SAMEPERIODLASTYEAR(Dim_Date[Date]), Dim_Date[Inpast] = TRUE)
PYTD_Quantity = CALCULATE([Quantity], SAMEPERIODLASTYEAR(Dim_Date[Date]), Dim_Date[Inpast] = TRUE)
PYTD_Sales = CALCULATE([Sales], SAMEPERIODLASTYEAR(Dim_Date[Date]), Dim_Date[Inpast] = TRUE)
```

##### **Year to Date (YTD)**
Tracks sales performance from the start of the year to the present:

```DAX
YTD_GrossProfit = TOTALYTD([Gross Profit], Fact_Sales[Date_Time])
YTD_Quantity = TOTALYTD([Quantity], Fact_Sales[Date_Time])
YTD_Sales = TOTALYTD([Sales], Fact_Sales[Date_Time])
```

### **Switching Measures for Dynamic Selection**
Used to dynamically calculate YTD and PYTD values based on slicer selection.

```DAX
S_PYTD =
VAR selected_value = SELECTEDVALUE(Slc_Values[Values])
VAR result = SWITCH(selected_value,
    "Sales", [PYTD_Sales],
    "Quantity", [PYTD_Quantity],
    "Gross Profit", [PYTD_GrossProfit],
    BLANK()
)
RETURN result

S_YTD =
VAR selected_value = SELECTEDVALUE(Slc_Values[Values])
VAR result = SWITCH(selected_value,
    "Sales", [YTD_Sales],
    "Quantity", [YTD_Quantity],
    "Gross Profit", [YTD_GrossProfit],
    BLANK()
)
RETURN result

YTD_vs_PYTD = [S_YTD] - [S_PYTD]
```
