# PostgreSQL JSONB Assignment: Flatten JSON Data from One Table to Another

## Question

You are given a PostgreSQL table that stores API response data in a JSONB column.

Your task is to:

- Store API response data in a JSONB column.
- Create a destination relational table.
- Extract fields from the JSONB data.
- Flatten the JSON data into rows and columns.
- Load the flattened data into another PostgreSQL table.
- Verify the inserted data.

---

## Step 1: Create Source Table (JSONB Storage)

```sql
CREATE TABLE api_data (
    id SERIAL PRIMARY KEY,
    payload JSONB
);
```

---

## Step 2: Insert JSON Data

```sql
INSERT INTO api_data (payload)
VALUES
(
'{
  "sale_id": 11,
  "rep_name": "Rohan Bajaj",
  "department": "Engineering",
  "city": "Hyderabad",
  "amount": 78000,
  "status": "open"
}'
),
(
'{
  "sale_id": 12,
  "rep_name": "Pooja Iyer",
  "department": "HR",
  "city": "Chennai",
  "amount": 44000,
  "status": "open"
}'
);
```

---

## Step 3: Verify Source Data

```sql
SELECT *
FROM api_data;
```

---

## Step 4: Create Destination Table

```sql
CREATE TABLE sales_flattened (
    sale_id INT,
    rep_name TEXT,
    department TEXT,
    city TEXT,
    amount NUMERIC,
    status TEXT
);
```

---

## Step 5: Flatten JSONB Data and Load into Destination Table

```sql
INSERT INTO sales_flattened (
    sale_id,
    rep_name,
    department,
    city,
    amount,
    status
)
SELECT
    (payload->>'sale_id')::INT       AS sale_id,
    payload->>'rep_name'             AS rep_name,
    payload->>'department'           AS department,
    payload->>'city'                 AS city,
    (payload->>'amount')::NUMERIC    AS amount,
    payload->>'status'               AS status
FROM api_data;
```

---

## Step 6: Verify Flattened Data

```sql
SELECT *
FROM sales_flattened;
```