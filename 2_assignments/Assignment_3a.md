# Assignment 3: Automated Data Ingestion Pipeline

## Problem Statement

A system receives CSV and JSON files every Monday from hundreds of different sources. Currently, data ingestion is done manually, which creates several problems:

- The process takes several hours.
- A single bad record can break the entire import.
- There is no proper error logging.
- Duplicate records can be inserted if the same file is processed twice.
- The process does not scale as the number of source files increases.

The goal is to design an automated PostgreSQL-based ingestion pipeline that can:

- Import both CSV and JSON files
- Handle bad data without crashing
- Log errors for debugging
- Avoid duplicate records
- Work reliably at high volume

---

## Solution Approach

To solve this problem, I designed a multi-stage ingestion pipeline using PostgreSQL staging tables, validation rules, and idempotent inserts.

Instead of directly loading source files into the final production table, the system follows a staged approach.

## Pipeline Flow

1. Receive CSV and JSON files
2. Load raw data into staging tables
3. Validate and clean the data
4. Insert valid records into production tables
5. Log invalid records into an error table
6. Track processed files to avoid duplicate ingestion

This approach ensures that bad records do not stop the entire pipeline and makes the system easier to debug and scale.

---

## Thought Process and Design Decisions

### 1. Why Use Staging Tables?

Directly importing external files into production tables is risky because:

- Source data may contain missing fields
- Data types may be incorrect
- One bad row can fail the entire transaction

To avoid this, I first load all raw data into staging tables.

**Benefits:**

- Fast bulk ingestion
- Easier validation
- Error isolation
- Safer production data

**Decision:** Use separate staging tables for CSV and JSON data.

---

### 2. How to Handle Both CSV and JSON?

CSV and JSON have different structures.

**CSV:**

- Fixed columns
- Structured format
- Best loaded using PostgreSQL `COPY`

**JSON:**

- Flexible schema
- Nested fields possible
- Best stored as `JSONB`

**Decision:** Use separate ingestion logic for each file type.

---

### 3. How to Prevent Duplicate Ingestion?

Sometimes automated systems retry jobs or the same file may be uploaded twice. Without protection, duplicate records can be inserted.

**Solution:**

- Track processed file names
- Use `ON CONFLICT DO NOTHING` during insert

**Decision:** Make the pipeline idempotent so rerunning it is safe.

---

### 4. How to Handle Bad Data Gracefully?

Bad records should not stop the entire import.

Instead:

- Detect invalid rows
- Store them in an error log table
- Continue processing valid rows

**Decision:** Use validation filters and error logging.

---

### 5. How to Scale for Hundreds of Files?

Manual row-by-row insertion is too slow.

To improve performance:

- Use `COPY` for bulk CSV loading
- Use `JSONB` for efficient JSON handling
- Index important columns
- Process files through scheduled automation

**Decision:** Optimize for batch-based high-volume ingestion.

---

## Database Schema

### Production Table

```sql
CREATE TABLE customer_data (
    customer_id BIGINT PRIMARY KEY,
    customer_name TEXT NOT NULL,
    email TEXT UNIQUE,
    source_file TEXT,
    created_at TIMESTAMP DEFAULT NOW()
);
```

This stores validated customer records.

### CSV Staging Table

```sql
CREATE TABLE staging_csv (
    customer_id TEXT,
    customer_name TEXT,
    email TEXT,
    source_file TEXT,
    imported_at TIMESTAMP DEFAULT NOW()
);
```

This table stores raw CSV data before validation.

### JSON Staging Table

```sql
CREATE TABLE staging_json (
    raw_data JSONB,
    source_file TEXT,
    imported_at TIMESTAMP DEFAULT NOW()
);
```

This table stores raw JSON payloads.

### Error Log Table

```sql
CREATE TABLE ingestion_errors (
    error_id BIGSERIAL PRIMARY KEY,
    source_file TEXT,
    raw_record JSONB,
    error_message TEXT,
    logged_at TIMESTAMP DEFAULT NOW()
);
```

This table stores invalid records and error messages.

### Processed Files Table

```sql
CREATE TABLE processed_files (
    file_name TEXT PRIMARY KEY,
    processed_at TIMESTAMP DEFAULT NOW()
);
```

This table tracks which files were already processed.

---

## SQL Commands: Pipeline Execution

### Step 1: Check Whether File Was Already Processed

```sql
SELECT 1
FROM processed_files
WHERE file_name = 'customers_monday.csv';
```

If the file exists, skip processing.

### Step 2: Load CSV File Into Staging Table

```sql
COPY staging_csv(customer_id, customer_name, email, source_file)
FROM '/imports/customers_monday.csv'
DELIMITER ','
CSV HEADER;
```

**Why `COPY`:** PostgreSQL `COPY` is optimized for high-speed bulk loading.

### Step 3: Load JSON File Into Staging Table

```sql
INSERT INTO staging_json(raw_data, source_file)
VALUES (
    '{
      "customer_id": 102,
      "customer_name": "Priya",
      "email": "priya@example.com"
    }'::jsonb,
    'customers_102.json'
);
```

### Step 4: Log Bad CSV Records Gracefully

Example: invalid `customer_id = ABC123`.

```sql
INSERT INTO ingestion_errors (
    source_file,
    raw_record,
    error_message
)
SELECT
    source_file,
    jsonb_build_object(
        'customer_id', customer_id,
        'customer_name', customer_name,
        'email', email
    ),
    'Invalid customer_id format'
FROM staging_csv
WHERE customer_id !~ '^[0-9]+$';
```

This logs the bad record instead of crashing.

### Step 5: Insert Valid CSV Records Into Production

```sql
INSERT INTO customer_data (
    customer_id,
    customer_name,
    email,
    source_file
)
SELECT
    customer_id::BIGINT,
    customer_name,
    email,
    source_file
FROM staging_csv
WHERE customer_id ~ '^[0-9]+$'
ON CONFLICT (customer_id) DO NOTHING;
```

`ON CONFLICT` prevents duplicate records.

### Step 6: Insert Valid JSON Records Into Production

```sql
INSERT INTO customer_data (
    customer_id,
    customer_name,
    email,
    source_file
)
SELECT
    (raw_data->>'customer_id')::BIGINT,
    raw_data->>'customer_name',
    raw_data->>'email',
    source_file
FROM staging_json
WHERE raw_data ? 'customer_id'
ON CONFLICT (customer_id) DO NOTHING;
```

### Step 7: Mark File As Processed

```sql
INSERT INTO processed_files(file_name)
VALUES ('customers_monday.csv')
ON CONFLICT DO NOTHING;
```

This prevents reprocessing.

---

## Example 1: Clean Import

### Input CSV

```text
101,Rahul,rahul@example.com
```

### Pipeline Behavior

- File loaded into `staging_csv`
- Validation passed
- Inserted into `customer_data`
- File marked as processed

### Final Result

```text
customer_id = 101
status = imported successfully
```

---

## Example 2: Gracefully Handled Error

### Input CSV

```text
ABC123,John,john@example.com
```

### Problem

`customer_id` should be numeric, but the value is invalid.

### Pipeline Behavior

- File loaded into `staging_csv`
- Validation failed
- Record inserted into `ingestion_errors`
- Production insert skipped
- Pipeline continues normally

### Error Log Entry

```text
source_file: customers_bad.csv
raw_record: {"customer_id":"ABC123","customer_name":"John"}
error_message: Invalid customer_id format
```

### Final Result

```text
customer_id = ABC123
status = rejected and logged
```

---

## Conclusion

This PostgreSQL-based ingestion pipeline solves the main problems of manual data import. It supports both CSV and JSON files, separates raw data from production data using staging tables, logs invalid records, prevents duplicate processing, and scales better for hundreds of weekly files.

By using staging tables, validation rules, `COPY`, `JSONB`, `ON CONFLICT DO NOTHING`, and processed file tracking, the pipeline becomes reliable, fault-tolerant, and safe to rerun.
