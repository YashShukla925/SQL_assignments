# Sales Table SQL Script

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
(1 ,'unnati' , 'IT' , 'kanpur' , 30000.50 , 'jan' , 'active' ),
(2 ,'priya' , 'ITI' , 'nagpur' , 340000.07 , 'feb' , 'active' ),
(3 ,'riya' , 'SALES' , 'durgapur' , 530000.04 , 'dec' , 'not-active' ),
(4 ,'diya' , 'FINANCE' , 'kaanakpur' , 306000.50 , 'march' , 'not-active' );
```

## View Data

```sql
SELECT * FROM sales;
```

## Transaction: Delete High Salary Records

> Delete users whose salary (amount) is greater than 20% of total salary.

```sql
BEGIN TRANSACTION;

DELETE FROM sales
WHERE amount > (
    SELECT SUM(amount) * 0.20 FROM sales
);

SELECT * FROM sales;

COMMIT;

-- If needed to undo changes before commit:
-- ROLLBACK;
```
