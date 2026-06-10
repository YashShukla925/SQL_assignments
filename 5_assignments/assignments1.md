# SQL Window Functions Practice

## Create Table

```sql
CREATE TABLE scores (
    student_id  INT PRIMARY KEY,
    name        VARCHAR(100),
    subject     VARCHAR(50),
    score       INT,
    exam_date   DATE,
    grade       VARCHAR(5)
);
```

## Insert Sample Data

```sql
INSERT INTO scores VALUES
(1,  'Amit',  'Maths',   92, '2026-01-15', 'A'),
(2,  'Sara',  'Maths',   85, '2026-01-15', 'B'),
(3,  'Ravi',  'Maths',   92, '2026-01-15', 'A'),
(4,  'Neha',  'Maths',   78, '2026-01-15', 'B'),
(5,  'Karan', 'Maths',   65, '2026-01-15', 'C'),
(6,  'Amit',  'Science', 88, '2026-02-10', 'A'),
(7,  'Sara',  'Science', 91, '2026-02-10', 'A'),
(8,  'Ravi',  'Science', 74, '2026-02-10', 'C'),
(9,  'Neha',  'Science', 88, '2026-02-10', 'A'),
(10, 'Karan', 'Science', 55, '2026-02-10', 'D'),
(11, 'Amit',  'English', 79, '2026-03-05', 'B'),
(12, 'Sara',  'English', 95, '2026-03-05', 'A'),
(13, 'Ravi',  'English', 82, '2026-03-05', 'B'),
(14, 'Neha',  'English', 79, '2026-03-05', 'B'),
(15, 'Karan', 'English', 88, '2026-03-05', 'A');
```

---

# Question 16: Running Total Per Student

## Problem Statement

Calculate the cumulative score obtained by each student across all exams in chronological order.

## Query

```sql
SELECT
    name,
    subject,
    score,
    exam_date,
    SUM(score) OVER (
        PARTITION BY name
        ORDER BY exam_date
    ) AS running_total
FROM scores
ORDER BY name, exam_date;
```

---

# Question 17: Moving Average (Last 2 Exams)

## Problem Statement

Calculate the moving average score for each student using the current exam and the previous exam.

## Query

```sql
SELECT
    name,
    subject,
    score,
    exam_date,
    AVG(score) OVER (
        PARTITION BY name
        ORDER BY exam_date
        ROWS BETWEEN 1 PRECEDING AND CURRENT ROW
    ) AS moving_avg_last_2_exams
FROM scores
ORDER BY name, exam_date;
```
---

