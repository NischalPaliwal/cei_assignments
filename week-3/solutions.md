# Assignment 3 Solutions

---

## Section 1: Setup

### Create `superstore_raw` Table

```sql
CREATE TABLE superstore_raw (
    row_id INT PRIMARY KEY,
    order_id VARCHAR(25) NOT NULL,
    order_date DATE NOT NULL,
    ship_date DATE NOT NULL,
    ship_mode VARCHAR(25) NOT NULL,
    customer_id VARCHAR(25) NOT NULL,
    customer_name VARCHAR(100) NOT NULL,
    segment VARCHAR(25) NOT NULL,
    country VARCHAR(50) NOT NULL,
    city VARCHAR(50) NOT NULL,
    state VARCHAR(50) NOT NULL,
    postal_code VARCHAR(15) NULL,
    region VARCHAR(25) NOT NULL,
    product_id VARCHAR(25) NOT NULL,
    category VARCHAR(25) NOT NULL,
    sub_category VARCHAR(25) NOT NULL,
    product_name VARCHAR(255) NOT NULL,
    sales DECIMAL(10, 4) NOT NULL,
    quantity INT NOT NULL,
    discount DECIMAL(4, 2) NOT NULL,
    profit DECIMAL(10, 4) NOT NULL
);
```

### Bulk Insert Source Data

```sql
BULK INSERT superstore_raw
FROM 'C:\Users\PALIW\Downloads\superstore_data\source.csv'
WITH (
    FORMAT = 'CSV',
    FIRSTROW = 2,
    FIELDTERMINATOR = ',',
    ROWTERMINATOR = '\n'
);
```

### Create Derived Tables

```sql
SELECT DISTINCT
    customer_id, 
    customer_name, 
    segment
INTO customers
FROM superstore_raw;
```

```sql
SELECT DISTINCT
    product_id,
    product_name,
    category,
    sub_category
INTO products
FROM superstore_raw;
```

```sql
SELECT DISTINCT
    order_id,
    customer_id,
    order_date,
    ship_date,
    ship_mode,
    sales,
    country,
    city,
    state,
    postal_code,
    region
INTO orders
FROM superstore_raw;
```

---

## Section 2: Queries

### Q1 — Orders with Above-Average Sales (Subquery)

```sql
SELECT *
FROM orders
WHERE sales > (SELECT AVG(sales) FROM orders);
```

<img width="1474" height="857" alt="Screenshot 2026-06-08 190241" src="https://github.com/user-attachments/assets/1fb33031-dff4-47e2-bc63-00ffd12a028a" />

---

### Q2 — Highest Sales Order per Customer (Subquery)

```sql
SELECT
    c.customer_id,
    c.customer_name,
    o.order_id,
    o.sales AS highest_sales
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id
WHERE o.sales = (
    SELECT MAX(sub.sales)
    FROM orders sub
    WHERE sub.customer_id = c.customer_id
);
```

<img width="1474" height="860" alt="Screenshot 2026-06-08 190319" src="https://github.com/user-attachments/assets/4783d6d8-2deb-4e14-9f67-74471d991653" />

---

### Q3 — Total Sales per Customer (CTE)

```sql
WITH joined_customers_orders AS (
    SELECT
        c.customer_id AS customer_id,
        c.customer_name AS customer_name,
        o.order_id AS order_id,
        o.sales AS sales
    FROM customers c
    JOIN orders o ON c.customer_id = o.customer_id
)

SELECT
    customer_id,
    customer_name,
    SUM(sales) AS total_sales
FROM joined_customers_orders
GROUP BY
    customer_id, 
    customer_name;
```

<img width="1476" height="865" alt="Screenshot 2026-06-08 192813" src="https://github.com/user-attachments/assets/0bb92350-f5ea-400e-a0a9-dbba39af98d2" />

---

### Q4 — Customers with Above-Average Total Sales (CTE + Subquery)

```sql
WITH customer_totals AS (
    SELECT
        c.customer_id,
        c.customer_name,
        SUM(o.sales) AS total_sales
    FROM customers c
    JOIN orders o ON c.customer_id = o.customer_id
    GROUP BY c.customer_id, c.customer_name
)

SELECT
    customer_id,
    customer_name,
    total_sales
FROM customer_totals
WHERE total_sales > (SELECT AVG(total_sales) FROM customer_totals);
```

<img width="1473" height="864" alt="Screenshot 2026-06-08 193557" src="https://github.com/user-attachments/assets/24298841-9279-4df9-bb55-9048719cc1fe" />

---

### Q5 — Rank Customers by Total Sales (Window Function)

```sql
WITH customer_totals AS (
    SELECT
        c.customer_id,
        c.customer_name,
        SUM(o.sales) AS total_sales
    FROM customers c
    JOIN orders o ON c.customer_id = o.customer_id
    GROUP BY c.customer_id, c.customer_name
)

SELECT
    RANK() OVER (ORDER BY total_sales DESC) AS ranking,
    customer_id,
    customer_name,
    total_sales
FROM customer_totals;
```

<img width="1476" height="858" alt="Screenshot 2026-06-08 195844" src="https://github.com/user-attachments/assets/e6f0c6b0-b136-414d-8fc4-53b12cd2efee" />

---

### Q6 — Row Numbers per Customer by Sales (Window Function + PARTITION BY)

```sql
SELECT
    ROW_NUMBER() OVER (
        PARTITION BY customer_id 
        ORDER BY sales DESC
    ) AS ranking,
    customer_id,
    customer_name,
    order_id,
    sales
FROM
    (
        SELECT
            c.customer_id,
            c.customer_name,
            o.order_id,
            o.sales
        FROM customers c
        JOIN orders o ON c.customer_id = o.customer_id
    ) subquery;
```

<img width="1472" height="746" alt="Screenshot 2026-06-08 201043" src="https://github.com/user-attachments/assets/7df7c6bd-b228-4188-9973-3a717d77c739" />

---

### Q7 — Top 3 Customers by Total Sales (Window Function + DENSE_RANK)

```sql
SELECT *
FROM
(
    SELECT
        DENSE_RANK() OVER (ORDER BY SUM(sales) DESC) AS ranking,
        customer_id,
        customer_name,
        SUM(sales) AS total_sales
    FROM
        (
            SELECT
                c.customer_id,
                c.customer_name,
                o.order_id,
                o.sales
            FROM customers c
            JOIN orders o ON c.customer_id = o.customer_id
        ) subquery1
    GROUP BY customer_id, customer_name
) subquery2
WHERE ranking <= 3;
```

<img width="1266" height="197" alt="Screenshot 2026-06-08 202302" src="https://github.com/user-attachments/assets/7db8eee2-b783-4162-a176-3c87ff40e4c2" />

---

### Final Combined Query (CTE + Window Function)

```sql
WITH customers_join_orders AS (
    SELECT
        c.customer_id,
        c.customer_name,
        o.order_id,
        o.sales
    FROM customers c
    JOIN orders o ON c.customer_id = o.customer_id
)

SELECT
    DENSE_RANK() OVER (ORDER BY SUM(sales) DESC) AS ranking,
    customer_id,
    customer_name,
    SUM(sales) AS total_sales
FROM customers_join_orders
GROUP BY customer_id, customer_name;
```

<img width="1037" height="627" alt="Screenshot 2026-06-08 202725" src="https://github.com/user-attachments/assets/598aecfc-dfb0-4891-baac-edd735a08105" />

---

## Section 3: Mini Project

### 1 — Top 5 Customers

```sql
SELECT
    TOP 5 ranking,
    *
FROM
(
    SELECT
        DENSE_RANK() OVER (ORDER BY SUM(sales) DESC) AS ranking,
        customer_id,
        customer_name,
        SUM(sales) AS total_sales
    FROM
        (
            SELECT
                c.customer_id,
                c.customer_name,
                o.order_id,
                o.sales
            FROM customers c
            JOIN orders o ON c.customer_id = o.customer_id
        ) subquery1
    GROUP BY customer_id, customer_name
) subquery2;
```

<img width="811" height="244" alt="Screenshot 2026-06-08 203214" src="https://github.com/user-attachments/assets/78ee6e75-c887-4143-b38d-ae76da892091" />

---

### 2 — Bottom 5 Customers

```sql
SELECT
    TOP 5 ranking,
    *
FROM
(
    SELECT
        DENSE_RANK() OVER (ORDER BY SUM(sales)) AS ranking,
        customer_id,
        customer_name,
        SUM(sales) AS total_sales
    FROM
        (
            SELECT
                c.customer_id,
                c.customer_name,
                o.order_id,
                o.sales
            FROM customers c
            JOIN orders o ON c.customer_id = o.customer_id
        ) subquery1
    GROUP BY customer_id, customer_name
) subquery2;
```

<img width="911" height="281" alt="Screenshot 2026-06-08 203243" src="https://github.com/user-attachments/assets/504a3195-7d0b-46c2-9b12-15d24ad67ac8" />

---

### 3 — Customers with Only One Order

```sql
SELECT
    c.customer_id,
    c.customer_name
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id
GROUP BY c.customer_id, c.customer_name
HAVING COUNT(order_id) = 1;
```

<img width="456" height="237" alt="Screenshot 2026-06-08 210250" src="https://github.com/user-attachments/assets/2f65601a-c6fe-43e0-8147-18c4ebeb80ab" />

---

### 4 — Customers with Above-Average Total Sales

```sql
WITH customer_totals AS (
    SELECT
        c.customer_id,
        c.customer_name,
        SUM(o.sales) AS total_sales
    FROM customers c
    JOIN orders o ON c.customer_id = o.customer_id
    GROUP BY c.customer_id, c.customer_name
)

SELECT
    customer_id,
    customer_name
FROM customer_totals
WHERE total_sales > (SELECT AVG(total_sales) FROM customer_totals);
```

<img width="1476" height="791" alt="Screenshot 2026-06-08 210416" src="https://github.com/user-attachments/assets/54852897-4e7f-4644-99a8-0b804732843d" />

---

### 5 — Highest Order Value per Customer

```sql
SELECT
    c.customer_id,
    c.customer_name,
    MAX(o.sales) AS highest_order_value
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id
GROUP BY c.customer_id, c.customer_name;
```

<img width="1474" height="779" alt="Screenshot 2026-06-08 210626" src="https://github.com/user-attachments/assets/eb5a29e2-cfff-41c0-bed6-ba67bbeec0d0" />
