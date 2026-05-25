# Assignment 1: String Functions and Regular Expressions

## Question 1: String Aggregation

Write a SQL query to show each department with all employee names combined into a single comma-separated list.

### Sample Table

```sql
CREATE TABLE department_employees (
    department_name TEXT,
    employee_name TEXT
);

INSERT INTO department_employees (department_name, employee_name)
VALUES
    ('Engineering', 'Amit Kumar'),
    ('Engineering', 'Priya Sharma'),
    ('Engineering', 'Rahul Verma'),
    ('HR', 'Neha Singh'),
    ('HR', 'Karan Mehta');
```

### Solution

```sql
SELECT
    department_name,
    STRING_AGG(employee_name, ', ' ORDER BY employee_name) AS employee_list
FROM department_employees
GROUP BY department_name
ORDER BY department_name;
```

### Expected Output

```text
department_name | employee_list
----------------+--------------------------------------
Engineering     | Amit Kumar, Priya Sharma, Rahul Verma
HR              | Karan Mehta, Neha Singh
```

---

## Question 2: Camel Case Conversion

Write a SQL query to convert normal text into camel case.

Example:

```text
employee full name -> employeeFullName
student roll number -> studentRollNumber
```

### Sample Table

```sql
CREATE TABLE text_values (
    id INT,
    input_text TEXT
);

INSERT INTO text_values (id, input_text)
VALUES
    (1, 'employee full name'),
    (2, 'student roll number'),
    (3, 'project start date');
```

### Solution

```sql
WITH words AS (
    SELECT
        id,
        word,
        word_position
    FROM text_values
    CROSS JOIN LATERAL regexp_split_to_table(lower(input_text), '\s+')
        WITH ORDINALITY AS split_words(word, word_position)
)
SELECT
    id,
    STRING_AGG(
        CASE
            WHEN word_position = 1 THEN word
            ELSE INITCAP(word)
        END,
        ''
        ORDER BY word_position
    ) AS camel_case_text
FROM words
GROUP BY id
ORDER BY id;
```

### Expected Output

```text
id | camel_case_text
---+------------------
1  | employeeFullName
2  | studentRollNumber
3  | projectStartDate
```

---

## Question 3: Replace Text Using Regular Expression

Write a SQL query using regular expression replacement to change:

- `amit kumar singh` into `amit kumar_singh`
- `amit kumar singh pandey` into `amit kumar_singh pandey`

Only the space between `kumar` and `singh` should be replaced with an underscore.

### Sample Table

```sql
CREATE TABLE person_names (
    id INT,
    full_name TEXT
);

INSERT INTO person_names (id, full_name)
VALUES
    (1, 'amit kumar singh'),
    (2, 'amit kumar singh pandey'),
    (3, 'rahul kumar singh');
```

### Solution

```sql
SELECT
    id,
    full_name,
    regexp_replace(
        full_name,
        '\m(amit kumar) (singh)\M',
        '\1_\2',
        'i'
    ) AS updated_name
FROM person_names
ORDER BY id;
```

### Expected Output

```text
id | full_name                 | updated_name
---+---------------------------+----------------------------
1  | amit kumar singh          | amit kumar_singh
2  | amit kumar singh pandey   | amit kumar_singh pandey
3  | rahul kumar singh         | rahul kumar singh
```
