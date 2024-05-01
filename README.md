
Data Analyst Project – Sales Management
=======================================

![](https://ningjoanne.files.wordpress.com/2022/09/sales-analysis_jo_overview-1.jpg?w=1024)

<div style="display: flex; justify-content: center;">
    <img src="https://ningjoanne.files.wordpress.com/2022/09/sql-project-ssms-2.png?w=763" alt="Image 1" style="width: 33%;">
    <img src="https://ningjoanne.files.wordpress.com/2022/09/sql-project-model-7.png?w=1024" alt="Image 2" style="width: 33%;">
    <img src="https://ningjoanne.files.wordpress.com/2022/09/sql-project-csv-6.png?w=784" alt="Image 3" style="width: 33%; height: auto;">
</div>





* * *

Business Request and User Stories
---------------------------------

This data analysis project is requested by a Sales Manager who would like to have a more interactive report to follow up on the overall sales.  
Based on the requests of the business, we would like to designate the “User Stories” to fulfill the demand and ensure the essential criteria were fully maintained for delivery.



* * *

| Role               | Request / Demand                                 | User Value                                                    | Acceptance Criteria                                      |
|--------------------|--------------------------------------------------|---------------------------------------------------------------|----------------------------------------------------------|
| Sales Manager      | A dashboard with an overview of online sales     | Can follow up on the best-selling product and figure out the loyal customers | A daily-updated Power BI dashboard.                     |
| Sales Representative | A detailed overview of Internet Sales per Customer | Can identify who are our SUPER customers and who are potential customers we can sell more | A Power BI dashboard with a “customer filter” function. |
| Sales Representative | A detailed overview of Internet Sales per Product  | Can identify the best/slowest sellers in the product aspect | A Power BI dashboard with a “product filter” function.  |
| Sales Manager      | A dashboard overview of internet sales            | Can check the total sales over time against budget          | A Power Bi dashboard with graphs and KPIs comparing sales v.s budget. |



Data Cleansing & Transformation (SQL)
-------------------------------------

The following tables are extracted from SQL to create the data model for fulfilling and analyzing the business demands based on the user stories.  
One of the data sources (Sales Budgets) is provided in Excel format and is connected to the data model in the later step of the process.

Below are the SQL statements and the necessary data transformation.

### **Dim\_Calendar :**

    --cleansed DIM_DateTable--
    SELECT 
      [DateKey], 
      [FullDateAlternateKey] AS Date,
      --[DayNumberOfWeek], 
      [EnglishDayNameOfWeek] AS Day,
      --,[SpanishDayNameOfWeek]
      --,[FrenchDayNameOfWeek]
      --,[DayNumberOfMonth]
      --,[DayNumberOfYear]
      [WeekNumberOfYear] AS Week, 
      [EnglishMonthName] AS Month,
      LEFT([EnglishMonthName],3) AS MonthShort,
      --,[SpanishMonthName]
      --,[FrenchMonthName]
      [MonthNumberOfYear] AS MonthNo, 
      [CalendarQuarter] AS Quarter, 
      [CalendarYear] AS Year
      --,[CalendarSemester]
      --,[FiscalQuarter]
      --,[FiscalYear]
      --,[FiscalSemester]
    FROM
      [AdventureWorksDW2019].[dbo].[DimDate]
      WHERE CalendarYear >=2019

### **Dim\_Customer :**

    -- Cleansed DIM_Customers Table --
    SELECT 
      c.customerkey AS CustomerKey, 
      --,[GeographyKey]
      --,[CustomerAlternateKey]
      --,[Title]
      c.firstname AS [First Name], 
      --,[MiddleName]
      c.lastname AS [Last Name], 
      c.firstname + ' ' + c.lastname AS [Full Name], 
      --combined First and Last name
      --,[NameStyle]
      --,[BirthDate]
      --,[MaritalStatus]
      --,[Suffix]
      CASE c.gender WHEN 'M' THEN 'Male' Else 'Female' END AS Gender, 
      --,[EmailAddress]
      --,[YearlyIncome]
      --,[TotalChildren]
      --,[NumberChildrenAtHome]
      --,[EnglishEducation]
      --,[SpanishEducation]
      --,[FrenchEducation]
      --,[EnglishOccupation]
      --,[SpanishOccupation]
      --,[FrenchOccupation]
      --,[HouseOwnerFlag]
      --,[NumberCarsOwned]
      --,[AddressLine1]
      --,[AddressLine2]
      --,[Phone]
      c.datefirstpurchase AS DateFirstPurchase, 
      --,[CommuteDistance]
      g.city AS [Customer City] -- Joined customer city from Geography Table
    FROM 
      dbo.DimCustomer AS c 
      LEFT JOIN dbo.dimgeography AS g ON g.geographykey = c.geographykey -- Joined in Customer City from Geography Table
    ORDER BY 
      Customerkey ASC

### **DIM\_Products :**

    -- cleansed DIM_Products Table --
    SELECT
    	p.[ProductKey],
    	p.[ProductAlternateKey] AS Product_Item_Code,
          --,[ProductSubcategoryKey]
          --,[WeightUnitMeasureCode]
          --,[SizeUnitMeasureCode]
        p.[EnglishProductName] AS [Product Name],
    	ps.EnglishProductSubcategoryName AS [Sub Category], --Joined in from Sub Category Table
        pc.EnglishProductCategoryName AS [Product Category], --Joined in from Category Table
    	--,[SpanishProductName]
     --   ,[FrenchProductName]
     --   ,[StandardCost]
     --   ,[FinishedGoodsFlag]
        p.[Color] AS [Product Color],
        --,[SafetyStockLevel]
        --,[ReorderPoint]
        --,[ListPrice]
        p.[Size] AS [Product Size],
        --,[SizeRange]
        --,[Weight]
        --,[DaysToManufacture]
        p.[ProductLine] AS [Product Line],
        --,[DealerPrice]
        --,[Class]
        --,[Style]
        p.[ModelName] AS [Product Model Name],
        --,[LargePhoto]
        p.[EnglishDescription] AS [Product Description],
        --,[FrenchDescription]
        --,[ChineseDescription]
        --,[ArabicDescription]
        --,[HebrewDescription]
        --,[ThaiDescription]
        --,[GermanDescription]
        --,[JapaneseDescription]
        --,[TurkishDescription]
        --,[StartDate]
        --,[EndDate]
        ISNULL(p.status,'OutDated') AS [Product Status]
      FROM [dbo].[DimProduct] AS p
      LEFT JOIN dbo.DimProductSubCategory AS ps
    	ON p.ProductSubcategoryKey = ps.ProductSubcategoryKey
      LEFT JOIN dbo.DimProductCategory AS pc
    	ON ps.ProductCategoryKey = pc.ProductCategoryKey
    ORDER BY p.ProductKey ASC

### **FACT\_Internet Sales:**

    -- cleansed FACT_InternetSales Table --
    SELECT 
      [ProductKey], 
      [OrderDateKey], 
      [DueDateKey], 
      [ShipDateKey], 
      [CustomerKey], 
      --,[PromotionKey]
      --,[CurrencyKey]
      --,[SalesTerritoryKey]
      [SalesOrderNumber], 
      --,[SalesOrderLineNumber]
      --,[RevisionNumber]
      --,[OrderQuantity]
      --,[UnitPrice]
      --,[ExtendedAmount]
      --,[UnitPriceDiscountPct]
      --,[DiscountAmount]
      --,[ProductStandardCost]
      --,[TotalProductCost]
      [SalesAmount] --,[TaxAmt]
      --,[Freight]
      --,[CarrierTrackingNumber]
      --,[CustomerPONumber]
      --,[OrderDate]
      --,[DueDate]
      --,[ShipDate]
    FROM 
      [AdventureWorksDW2019].[dbo].[FactInternetSales] 
    WHERE 
      LEFT(OrderDateKey, 4) >= YEAR(GETDATE())-2 --Ensure data only includes two years of date from extraction.
    ORDER BY 
      OrderDateKey ASC

* * *

Data Model
----------

After cleansing data and rearranging the tables, below is the structure of the data model which is imported into Power BI for further visual analysis.  
This data model also shows how Sales Budget in Excel format (FACT\_Budget) has been connected to the Internet Sales table (FACT\_InternetSales) as well as the connections among necessary Dimension Tables.

![](https://ningjoanne.files.wordpress.com/2022/09/sql-project-model-edited.png)

* * *

Sales Management Dashboard
--------------------------

The final Sales Management Report comes with a Sales Overview Dashboard on the first page with another two pages focusing on combining tables from the necessary details and visualizing the sales over time by customers and by products.

**Click the picture below to open the dashboard and try it out** !!

[![](https://ningjoanne.files.wordpress.com/2022/09/sql-project-overview-1-2.png?w=1024)](https://app.powerbi.com/view?r=eyJrIjoiYzQwNDlkMmItMTRmZS00ZTVhLTk2MjAtMmYwNTJiODkxNDRhIiwidCI6Ijg5NGIzZjE2LTg2MTMtNDljZS05MzZjLTc5ZGYxMjY2M2ViNiJ9)

* * *

References:

[www.youtube.com/@iamaliahmad](http://www.youtube.com/@iamaliahmad)

