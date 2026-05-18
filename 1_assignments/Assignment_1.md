# Assignment 1: Student Scholarship Amount Calculation

## Problem Statement

A college stores student payment records in one table and scholarship details in another table. Each student has a base amount that they need to pay. Some students may receive one or more scholarships.

The task is to:

1. Create a `student` table to store student payment details.
2. Create a `scholarship` table to store scholarship records.
3. Insert sample student and scholarship data.
4. Write a SQL query to calculate the final payable amount for each student after subtracting active scholarships.

If the scholarship amount is greater than the base amount, the final payable amount should not become negative. It should be shown as `0`.

---

## Solution Approach

To solve this problem, I used two related tables:

- `student` table for student details and base payment amount
- `scholarship` table for scholarship details linked to each student

The `scholarship` table uses `student_id` as a foreign key, which references the `student_id` column in the `student` table.

The final amount is calculated as:

```text
final_amount = base_amount - total_active_scholarship
```

To avoid negative final amounts, the `GREATEST()` function is used.

---

## Schema of Tables

### Student Table

```sql
CREATE TABLE student (
    id SERIAL PRIMARY KEY,
    student_id VARCHAR(50) NOT NULL UNIQUE,
    student_name VARCHAR(255) NOT NULL,
    transaction_id VARCHAR(100) NOT NULL UNIQUE,
    amount NUMERIC(12,2) NOT NULL DEFAULT 0,
    status VARCHAR(50) NOT NULL DEFAULT 'pending',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### Scholarship Table

```sql
CREATE TABLE scholarship (
    id SERIAL PRIMARY KEY,
    student_id VARCHAR(50) NOT NULL,
    scholarship_amount NUMERIC(12,2) NOT NULL,
    status VARCHAR(50) DEFAULT 'active',
    granted_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

    FOREIGN KEY (student_id)
    REFERENCES student(student_id)
);
```

---

## Insert Sample Data

### Insert Student Records

```sql
INSERT INTO student
(student_id, student_name, transaction_id, amount, status)
VALUES
('STU001', 'qqw', 'TXN001', 5000, 'success'),
('STU002', 'swe', 'TXN002', 7000, 'success'),
('STU003', 'ttx', 'TXN003', 6000, 'success'),
('STU004', 'nhw', 'TXN004', 8000, 'success');
```

### Insert Scholarship Records

```sql
INSERT INTO scholarship
(student_id, scholarship_amount, status)
VALUES
('STU001', 1500, 'active'),
('STU002', 2000, 'active'),
('STU002', 500, 'active'),
('STU004', 9000, 'active');
```

---

## Query for Final Amount

```sql
SELECT
    s.student_name,
    s.amount AS base_amount,

    COALESCE(SUM(sc.scholarship_amount), 0)
        AS scholarship,

    GREATEST(
        s.amount - COALESCE(SUM(sc.scholarship_amount), 0),
        0
    ) AS final_amount

FROM student s
LEFT JOIN scholarship sc
    ON s.student_id = sc.student_id
   AND sc.status = 'active'

GROUP BY
    s.student_name,
    s.amount;
```

---

## Explanation of Query

### 1. LEFT JOIN

```sql
LEFT JOIN scholarship sc
    ON s.student_id = sc.student_id
   AND sc.status = 'active'
```

`LEFT JOIN` is used so that all students are included in the result, even if they do not have any scholarship.

The condition `sc.status = 'active'` ensures that only active scholarships are considered.

### 2. SUM

```sql
SUM(sc.scholarship_amount)
```

This calculates the total scholarship amount for each student. A student can have more than one scholarship, so `SUM()` is required.

### 3. COALESCE

```sql
COALESCE(SUM(sc.scholarship_amount), 0)
```

If a student has no scholarship, `SUM()` returns `NULL`. `COALESCE()` converts that `NULL` value into `0`.

### 4. GREATEST

```sql
GREATEST(
    s.amount - COALESCE(SUM(sc.scholarship_amount), 0),
    0
)
```

This prevents the final amount from becoming negative.

For example, student `STU004` has:

```text
base_amount = 8000
scholarship = 9000
```

Without `GREATEST()`, the final amount would be:

```text
8000 - 9000 = -1000
```

Using `GREATEST()`, the final amount becomes:

```text
0
```

---

## Expected Output

| student_name | base_amount | scholarship | final_amount |
|---|---:|---:|---:|
| qqw | 5000.00 | 1500.00 | 3500.00 |
| swe | 7000.00 | 2500.00 | 4500.00 |
| ttx | 6000.00 | 0.00 | 6000.00 |
| nhw | 8000.00 | 9000.00 | 0.00 |

---

## Conclusion

This assignment demonstrates how to create related tables using primary keys and foreign keys, insert sample records, and write a query using `LEFT JOIN`, `SUM()`, `COALESCE()`, and `GREATEST()` to calculate the final payable amount after scholarships.

The query correctly handles students with no scholarship, students with multiple scholarships, and students whose scholarship amount is greater than their base amount.
