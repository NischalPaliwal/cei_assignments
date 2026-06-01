# Week 2 Assignment Solutions

## 1. Load Dataset into SQL Database

```sql
CREATE DATABASE TestDB;

USE TestDB;


CREATE TABLE Orders (
    RowID         INT,
    OrderID       VARCHAR(20),
    OrderDate     DATE,
    ShipDate      DATE,
    ShipMode      VARCHAR(30),
    CustomerID    VARCHAR(20),
    CustomerName  VARCHAR(100),
    Segment       VARCHAR(30),
    Country       VARCHAR(50),
    City          VARCHAR(50),
    [State]       VARCHAR(50),
    PostalCode    VARCHAR(10),
    Region        VARCHAR(20),
    ProductID     VARCHAR(20),
    Category      VARCHAR(30),
    SubCategory   VARCHAR(30),
    ProductName   VARCHAR(255),
    Sales         DECIMAL(10,2),
    Quantity      INT,
    Discount      DECIMAL(4,2),
    Profit        DECIMAL(10,2)
);
```

Import the CSV file.

```sql
BULK INSERT Orders
FROM 'C:\Data\superstore.csv'
WITH (
    FORMAT           = 'CSV',
    FIRSTROW         = 2,
    FIELDTERMINATOR  = ',',
    ROWTERMINATOR    = '\n',
    TABLOCK
);
```

---

## 2. Explore the Table

**View schema**

```sql
SELECT
    COLUMN_NAME,
    DATA_TYPE,
    IS_NULLABLE
FROM INFORMATION_SCHEMA.COLUMNS
WHERE TABLE_NAME = 'Orders'
ORDER BY ORDINAL_POSITION;
```

**Sample data**

```sql
SELECT TOP 10 * FROM Orders;
```

**Row count and date range**

```sql
SELECT COUNT(*) AS TotalRows FROM Orders;

SELECT
    MIN(OrderDate) AS EarliestOrder,
    MAX(OrderDate) AS LatestOrder
FROM Orders;
```

**Distinct counts**

```sql
SELECT
    COUNT(DISTINCT CustomerID)  AS UniqueCustomers,
    COUNT(DISTINCT ProductID)   AS UniqueProducts,
    COUNT(DISTINCT Category)    AS Categories,
    COUNT(DISTINCT SubCategory) AS SubCategories
FROM Orders;
```

---

## 3. Apply WHERE Filters

**Filter by region**

```sql
SELECT OrderID, CustomerName, Region, Sales, Profit
FROM Orders
WHERE Region = 'West'
ORDER BY Sales DESC;
```

**Filter by category**

```sql
SELECT OrderID, ProductName, SubCategory, Sales, Profit
FROM Orders
WHERE Category = 'Technology';
```

**Filter by date range**

```sql
SELECT OrderID, OrderDate, CustomerName, Sales
FROM Orders
WHERE OrderDate BETWEEN '2017-01-01' AND '2017-12-31'
ORDER BY OrderDate;
```

**Filter by sales amount**

```sql
SELECT OrderID, CustomerName, Category, Sales, Profit
FROM Orders
WHERE Sales > 1000
ORDER BY Sales DESC;
```

**Combined filters**

```sql
SELECT OrderID, CustomerName, ProductName, Sales, Profit
FROM Orders
WHERE Category = 'Technology'
  AND Region   = 'West'
  AND Profit   > 0
ORDER BY Profit DESC;
```

---

## 4. GROUP BY Aggregations

**Sales by region**

```sql
SELECT
    Region,
    COUNT(*)             AS TotalOrders,
    SUM(Sales)           AS TotalSales,
    SUM(Profit)          AS TotalProfit,
    ROUND(AVG(Sales), 2) AS AvgOrderSales
FROM Orders
GROUP BY Region
ORDER BY TotalSales DESC;
```

**Sales by category**

```sql
SELECT
    Category,
    SUM(Sales)              AS TotalSales,
    SUM(Profit)             AS TotalProfit,
    ROUND(AVG(Discount), 2) AS AvgDiscount,
    SUM(Quantity)           AS TotalQuantity
FROM Orders
GROUP BY Category
ORDER BY TotalSales DESC;
```

**Sales by sub-category**

```sql
SELECT
    SubCategory,
    SUM(Sales)  AS TotalSales,
    SUM(Profit) AS TotalProfit,
    COUNT(*)    AS OrderCount
FROM Orders
GROUP BY SubCategory
ORDER BY TotalProfit DESC;
```

**Average sales by segment**

```sql
SELECT
    Segment,
    COUNT(DISTINCT CustomerID) AS Customers,
    ROUND(AVG(Sales), 2)       AS AvgSalesPerOrder,
    SUM(Sales)                 AS TotalSales
FROM Orders
GROUP BY Segment
ORDER BY TotalSales DESC;
```

---

## 5. Sort and Limit Results

**Top 10 products by sales**

```sql
SELECT TOP 10
    ProductName,
    SUM(Sales)  AS TotalSales,
    SUM(Profit) AS TotalProfit
FROM Orders
GROUP BY ProductName
ORDER BY TotalSales DESC;
```

**Top 5 most profitable sub-categories**

```sql
SELECT TOP 5
    SubCategory,
    SUM(Profit) AS TotalProfit
FROM Orders
GROUP BY SubCategory
ORDER BY TotalProfit DESC;
```

**Bottom 5 sub-categories by profit**

```sql
SELECT TOP 5
    SubCategory,
    SUM(Profit) AS TotalProfit
FROM Orders
GROUP BY SubCategory
ORDER BY TotalProfit ASC;
```

**Top 10 customers by revenue**

```sql
SELECT TOP 10
    CustomerID,
    CustomerName,
    Segment,
    SUM(Sales)              AS TotalRevenue,
    COUNT(DISTINCT OrderID) AS TotalOrders
FROM Orders
GROUP BY CustomerID, CustomerName, Segment
ORDER BY TotalRevenue DESC;
```

---

## 6. Solve Use Cases

**Monthly sales trends**

```sql
SELECT
    YEAR(OrderDate)             AS [Year],
    MONTH(OrderDate)            AS [Month],
    FORMAT(OrderDate, 'MMM')    AS MonthName,
    SUM(Sales)                  AS MonthlySales,
    SUM(Profit)                 AS MonthlyProfit
FROM Orders
GROUP BY YEAR(OrderDate), MONTH(OrderDate), FORMAT(OrderDate, 'MMM')
ORDER BY [Year], [Month];
```

**Year-over-year growth**

```sql
WITH YearlySales AS (
    SELECT
        YEAR(OrderDate) AS [Year],
        SUM(Sales)      AS TotalSales
    FROM Orders
    GROUP BY YEAR(OrderDate)
)
SELECT
    [Year],
    TotalSales,
    LAG(TotalSales) OVER (ORDER BY [Year]) AS PrevYearSales,
    ROUND(
        (TotalSales - LAG(TotalSales) OVER (ORDER BY [Year]))
        / LAG(TotalSales) OVER (ORDER BY [Year]) * 100, 1
    ) AS GrowthPct
FROM YearlySales;
```

**Top customer per region**

```sql
WITH CustomerSales AS (
    SELECT
        Region,
        CustomerName,
        SUM(Sales) AS TotalSales,
        RANK() OVER (PARTITION BY Region ORDER BY SUM(Sales) DESC) AS Rnk
    FROM Orders
    GROUP BY Region, CustomerName
)
SELECT Region, CustomerName, TotalSales
FROM CustomerSales
WHERE Rnk = 1;
```

**Find duplicate order line items**

```sql
SELECT
    OrderID, ProductID, CustomerID, OrderDate, Sales, Quantity,
    COUNT(*) AS DuplicateCount
FROM Orders
GROUP BY OrderID, ProductID, CustomerID, OrderDate, Sales, Quantity
HAVING COUNT(*) > 1
ORDER BY DuplicateCount DESC;
```

---

## 7. Validate Results

**Total row count**

```sql
SELECT COUNT(*) AS TotalRows FROM Orders;
```

**Check for NULL values**

```sql
SELECT
    SUM(CASE WHEN OrderID IS NULL THEN 1 ELSE 0 END) AS NullOrderID,
    SUM(CASE WHEN CustomerID IS NULL THEN 1 ELSE 0 END) AS NullCustomerID,
    SUM(CASE WHEN Sales IS NULL THEN 1 ELSE 0 END) AS NullSales,
    SUM(CASE WHEN Profit IS NULL THEN 1 ELSE 0 END) AS NullProfit,
    SUM(CASE WHEN OrderDate IS NULL THEN 1 ELSE 0 END) AS NullOrderDate
FROM Orders;