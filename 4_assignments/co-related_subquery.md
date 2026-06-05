# Assignment 4: Co-related Subquery

## Table Creation

```sql
CREATE TABLE sales (
    sale_id    INT PRIMARY KEY,
    rep_name   VARCHAR(100),
    department VARCHAR(50),
    city       VARCHAR(50),
    amount     DECIMAL(10,2),
    month      VARCHAR(20),
    status     VARCHAR(20)
);
```

## Insert Data

```sql
INSERT INTO sales VALUES
(1, 'unnati', 'IT', 'kanpur', 30000.50, 'jan', 'active'),
(2, 'priya', 'ITI', 'nagpur', 340000.07, 'feb', 'active'),
(3, 'riya', 'SALES', 'durgapur', 530000.04, 'dec', 'not-active'),
(4, 'diya', 'FINANCE', 'kaanakpur', 306000.50, 'march', 'not-active');
```

## View Table

```sql
SELECT * FROM sales;
```

## Problem Statement

Find sales representatives whose sales amount is greater than the average sales amount of their own department.

## Query

```sql
SELECT
    s1.sale_id,
    s1.rep_name,
    s1.department,
    s1.amount
FROM sales s1
WHERE s1.amount > (
    SELECT AVG(s2.amount)
    FROM sales s2
    WHERE s2.department = s1.department
);
```
