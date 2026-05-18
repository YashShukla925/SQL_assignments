# Assignment 3-b: Automated Weekly Report Export from PostgreSQL

## Problem Statement

A weekly report needs to be exported automatically from a PostgreSQL database every Friday. Right now, this is done manually, which creates some problems:

- Sometimes the report export is forgotten.
- Manual work takes extra time.
- There is no proper way to know if the export failed.
- If the same file name is used, older reports can get overwritten.

The goal is to automate this process so that the report is exported every Friday without manual intervention. The system should also create a unique file name using a timestamp and handle failures properly.

---

## Solution Approach

To solve this problem, I designed an automated export process using PostgreSQL.

The main idea is:

1. Export the required table into a CSV file.
2. Generate a timestamp-based file name so every report is saved separately.
3. Store export logs to track success or failure.
4. Handle errors automatically without stopping future runs.
5. Schedule the script to run every Friday.

This makes the whole process reliable and removes dependency on manual work.

---

## Thought Process and Design Decisions

### 1. Why Automate the Report Export?

Manual processes can easily be missed, especially if someone forgets to run the command. Since this report is needed every week, automation is a better option.

**Benefits:**

- Report is always generated on time.
- No manual effort is required.
- There is less chance of human error.

**Decision:** Automate the report generation.

---

### 2. Why Use PostgreSQL COPY Command?

PostgreSQL provides a built-in command for exporting data:

```sql
COPY table_name TO 'file.csv' CSV HEADER;
```

This is useful because:

- It is fast.
- It is built directly into PostgreSQL.
- It is easy to use.
- It works well for exporting large tables.

**Decision:** Use `COPY TO` for exporting the report.

---

### 3. Why Use Timestamp in File Name?

If every export uses the same name, such as:

```text
weekly_report.csv
```

then the old report will be replaced.

Instead, adding a timestamp creates unique files like:

```text
weekly_report_20260515_090000.csv
```

**Benefits:**

- Old reports are preserved.
- Reports are easy to identify by date and time.
- Accidental overwriting is avoided.

**Decision:** Use the current date and time in the file name.

---

### 4. How to Handle Failures?

Sometimes export can fail because of:

- Wrong file path
- Missing folder permissions
- Database issue
- Disk space problem

The system should not stop completely if this happens.

Instead:

- Catch the error.
- Save the error message in logs.
- Continue normal execution next week.

**Decision:** Use exception handling and logging.

---

### 5. How to Run Every Friday Automatically?

Since PostgreSQL does not schedule jobs by itself, a scheduler is needed.

A good option is `pg_cron`, which allows running PostgreSQL commands on a schedule.

**Decision:** Use `pg_cron` to run the export every Friday.

---

## Database Schema

### Report Table

This is the table that needs to be exported.

```sql
CREATE TABLE sales_report (
    order_id BIGINT,
    customer_name TEXT,
    total_amount NUMERIC(10,2),
    order_date DATE
);
```

### Export Log Table

This table stores whether the export succeeded or failed.

```sql
CREATE TABLE export_logs (
    export_id BIGSERIAL PRIMARY KEY,
    file_name TEXT,
    export_status TEXT,
    error_message TEXT,
    exported_at TIMESTAMP DEFAULT NOW()
);
```

---

## SQL Script

### Step 1: Create Export Procedure

```sql
CREATE OR REPLACE PROCEDURE export_weekly_report()
LANGUAGE plpgsql
AS $$
DECLARE
    file_name TEXT;
BEGIN
    -- Generate timestamped file name
    file_name :=
        '/exports/weekly_report_' ||
        TO_CHAR(NOW(), 'YYYYMMDD_HH24MISS') ||
        '.csv';

    BEGIN
        -- Export report
        EXECUTE format(
            'COPY sales_report TO %L WITH CSV HEADER',
            file_name
        );

        -- Log success
        INSERT INTO export_logs (
            file_name,
            export_status
        )
        VALUES (
            file_name,
            'SUCCESS'
        );

    EXCEPTION WHEN OTHERS THEN
        -- Log failure
        INSERT INTO export_logs (
            file_name,
            export_status,
            error_message
        )
        VALUES (
            file_name,
            'FAILED',
            SQLERRM
        );
    END;
END;
$$;
```

### Step 2: Test the Procedure Manually

```sql
CALL export_weekly_report();
```

This is useful to verify that everything works before scheduling it.

### Step 3: Schedule Automatic Run Every Friday

```sql
SELECT cron.schedule(
    'weekly_report_export',
    '0 9 * * 5',
    $$CALL export_weekly_report();$$
);
```

### Schedule Meaning

```text
0 9 * * 5
```

This means:

- `0` minutes
- `9 AM`
- Every Friday

So the report will automatically export every Friday at 9:00 AM.

---

## Example 1: Successful Test Run

Suppose the table contains:

| order_id | customer_name | total_amount | order_date |
|---|---|---:|---|
| 101 | Rahul | 2500.00 | 2026-05-15 |
| 102 | Priya | 1800.00 | 2026-05-15 |

### Generated File

```text
/exports/weekly_report_20260515_090000.csv
```

### CSV Output

```csv
order_id,customer_name,total_amount,order_date
101,Rahul,2500.00,2026-05-15
102,Priya,1800.00,2026-05-15
```

### Log Entry

```text
file_name: /exports/weekly_report_20260515_090000.csv
export_status: SUCCESS
error_message: NULL
```

### Result

The report was exported successfully.

---

## Example 2: Gracefully Handled Failure

Suppose the export folder does not exist:

```text
/exports/
```

PostgreSQL may throw an error like:

```text
could not open file
```

In this case:

- Export fails.
- Error is caught automatically.
- Failure is logged in `export_logs`.
- Next week's run will still happen normally.

### Log Entry

```text
file_name: /exports/weekly_report_20260515_090000.csv
export_status: FAILED
error_message: could not open file
```

### Result

The system does not crash and the issue can be checked later.

---

## Conclusion

This automated PostgreSQL export process solves the problem of manual weekly report generation. It uses `COPY TO` for fast CSV export, timestamp-based file names to prevent overwriting, an export log table to track success and failure, exception handling for reliability, and `pg_cron` to schedule the job every Friday.

The result is a repeatable, reliable, and easy-to-monitor weekly reporting system.
