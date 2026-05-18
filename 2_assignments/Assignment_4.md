# Assignment 4: Collaborative Schema Design + ER Diagram

## Problem Statement

As a team, we designed a PostgreSQL database for a company management system. The database stores information about departments, employees, clients, projects, salaries, tasks, and attendance.

Each intern owns one table. My assigned table is the `department` table.

The goal of my table is to store department details such as department name, manager, location, and creation time. This table connects with the `employee` table and the `project` table so that employees and projects can be grouped department-wise.

---

## Table Owner

**Table Name:** `department`

**Purpose:** This table stores all departments in the company, such as Engineering, Human Resources, Finance, Sales, and Marketing.

---

## Department Table Design

```sql
CREATE TABLE department (
    department_id SERIAL PRIMARY KEY,
    department_name VARCHAR(100) UNIQUE NOT NULL,
    manager_id INT,
    location VARCHAR(100),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

---

## Column Details and Constraint Justification

| Column Name | Data Type | Constraint | Justification |
|---|---|---|---|
| `department_id` | `SERIAL` | `PRIMARY KEY` | Automatically generates a unique ID for every department. This helps identify each department clearly. |
| `department_name` | `VARCHAR(100)` | `UNIQUE NOT NULL` | Department name is required and should not be repeated. For example, there should not be two departments named Engineering. |
| `manager_id` | `INT` | `FOREIGN KEY` | Stores the employee ID of the department manager. It links the department table with the employee table. |
| `location` | `VARCHAR(100)` | Optional | Stores the department location, such as Delhi Office or Mumbai Branch. It is optional because some departments may work remotely. |
| `created_at` | `TIMESTAMP` | `DEFAULT CURRENT_TIMESTAMP` | Automatically stores when the department record was created. |

---

## Foreign Key Relationship

The `department` table is linked with the `employee` table using `manager_id`.

```sql
ALTER TABLE department
ADD CONSTRAINT fk_department_manager
FOREIGN KEY (manager_id)
REFERENCES employee(employee_id);
```

### Relationship Explanation

- One department can have one manager.
- The manager must be an existing employee.
- This avoids storing invalid manager IDs in the department table.

The `employee` table also connects back to `department` through `department_id`, which means each employee belongs to one department.

---

## Realistic Dummy Data

```sql
INSERT INTO department (
    department_name,
    manager_id,
    location
)
VALUES
    ('Engineering', 1, 'Delhi Office'),
    ('Human Resources', 2, 'Mumbai Branch'),
    ('Finance', 3, 'Bangalore Office'),
    ('Sales', 4, 'Pune Office'),
    ('Marketing', 5, 'Remote');
```

Note: The `manager_id` values should already exist in the `employee` table before adding the foreign key or inserting manager-linked department records.

---

## Sample Business Query

This query shows department-wise project and employee information by joining all major business tables.

```sql
SELECT
    d.department_name,
    d.location,
    e.first_name || ' ' || e.last_name AS employee_name,
    p.project_name,
    c.client_name,
    t.task_name,
    s.base_salary,
    a.attendance_date,
    a.status AS attendance_status
FROM department d
JOIN employee e
    ON e.department_id = d.department_id
JOIN project p
    ON p.department_id = d.department_id
JOIN clients c
    ON c.client_id = p.client_id
JOIN tasks t
    ON t.employee_id = e.employee_id
    AND t.project_id = p.project_id
JOIN salary s
    ON s.employee_id = e.employee_id
JOIN attendance a
    ON a.employee_id = e.employee_id
ORDER BY
    d.department_name,
    employee_name;
```

### Query Purpose

This query helps the company understand which employees are working on which projects, which clients those projects belong to, their salary details, and attendance status department-wise.

---

## ER Diagram

ER Diagram Link:

https://dbdiagram.io/d/69d1a19a0f7c9ef2c07dbd29

---

## Conclusion

The `department` table is an important parent table in this company database. It organizes employees and projects by department and also stores manager information. By using primary key, unique, not null, default, and foreign key constraints, the table becomes accurate, connected, and reliable for business queries.
