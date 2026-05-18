# Assignment 1: Transaction Management at Scale

## Problem Statement

A nightly billing job needs to update 1 million records in a PostgreSQL database. Last week, the job crashed midway, and because there was no fault recovery mechanism, the entire process had to restart from the beginning. This caused wasted time and unnecessary database load.

The goal is to design a PostgreSQL solution that updates records efficiently, handles failures gracefully, and ensures that the job can resume from the last successful point instead of restarting from zero.

## Solution Approach

To solve this problem, I designed a fault-tolerant batch processing system using PostgreSQL transactions. Instead of updating all 1 million records in one large transaction, records are processed in smaller batches. After each successful batch, the progress is stored in a checkpoint table.

If the system crashes, the billing job reads the saved checkpoint and resumes from that point.

The main techniques used are:

1. Batch processing
2. Checkpointing
3. Idempotent updates
4. Transaction commit after each batch
5. Exception handling
6. Primary key range scanning

## Database Schema

### Main Billing Table

```sql
CREATE TABLE customer_billing (
    customer_id BIGINT PRIMARY KEY,
    amount_due NUMERIC(10,2),
    billed BOOLEAN DEFAULT FALSE,
    billing_date TIMESTAMP
);
```

This table stores billing records. The `billed` column indicates whether a customer has already been processed.

### Checkpoint Table

```sql
CREATE TABLE billing_job_checkpoint (
    job_name TEXT PRIMARY KEY,
    last_processed_id BIGINT,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

This table stores the last successfully processed customer ID so the job can resume after failure.

## PostgreSQL Stored Procedure

```sql
CREATE OR REPLACE PROCEDURE run_nightly_billing()
LANGUAGE plpgsql
AS $$
DECLARE
    batch_size INT := 10000;
    start_id BIGINT;
    max_id BIGINT;
    current_end BIGINT;
BEGIN
    -- Read checkpoint
    SELECT COALESCE(last_processed_id, 0)
    INTO start_id
    FROM billing_job_checkpoint
    WHERE job_name = 'nightly_billing';

    -- Initialize checkpoint on first run
    IF NOT FOUND THEN
        INSERT INTO billing_job_checkpoint(job_name, last_processed_id)
        VALUES ('nightly_billing', 0);

        start_id := 0;
    END IF;

    -- Get maximum customer ID
    SELECT MAX(customer_id)
    INTO max_id
    FROM customer_billing;

    -- Process data in batches
    WHILE start_id < max_id LOOP
        current_end := start_id + batch_size;

        BEGIN
            -- Update current batch
            UPDATE customer_billing
            SET billed = TRUE,
                billing_date = NOW()
            WHERE customer_id > start_id
              AND customer_id <= current_end
              AND billed = FALSE;

            -- Save progress
            UPDATE billing_job_checkpoint
            SET last_processed_id = current_end,
                updated_at = NOW()
            WHERE job_name = 'nightly_billing';

            -- Commit current batch
            COMMIT;

            -- Move to next batch
            start_id := current_end;

        EXCEPTION WHEN OTHERS THEN
            ROLLBACK;
            RAISE NOTICE 'Batch failed at customer_id %', start_id;
            EXIT;
        END;
    END LOOP;
END;
$$;
```

To execute this procedure:

```sql
CALL run_nightly_billing();
```

## Thought Process and Design Decisions

### 1. Avoiding One Large Transaction

Initially, updating all 1 million rows in a single SQL statement seems simple. However, in PostgreSQL, large transactions can create several problems:

- Long row locks
- High memory usage
- Large WAL (Write Ahead Log) generation
- Expensive rollback
- Entire progress lost if the process crashes

Because of these issues, using one large transaction is not efficient or reliable.

**Decision:** I chose batch-based transaction processing instead of a single large transaction.

### 2. Using Batch Processing

I divided the total workload into batches of 10,000 rows.

**Reason:**

- Smaller transactions complete faster
- Locks are held for less time
- PostgreSQL handles smaller WAL writes more efficiently
- Easier recovery after failure

**Calculation:**

```text
1,000,000 records / 10,000 per batch = 100 batches
```

If the system crashes after batch 60, only the remaining 40 batches need processing.

**Decision:** Set `batch_size = 10000`.

### 3. Adding Checkpointing

A checkpoint table stores the last successfully processed customer ID.

**Example:**

```text
last_processed_id = 540000
```

Then after restart, processing resumes from:

```text
customer_id = 540001
```

Without checkpointing, the job would always restart from the beginning. Checkpointing saves time and avoids repeating completed work.

**Decision:** Update the checkpoint table after every successful batch.

### 4. Making Updates Idempotent

The update query includes this condition:

```sql
AND billed = FALSE
```

This ensures that already processed records are skipped automatically.

**Benefits:**

- Safe to rerun the procedure
- Prevents duplicate billing
- Maintains data consistency

**Decision:** Use idempotent updates so retries are safe.

### 5. Committing After Each Batch

After every successful batch, the transaction is committed:

```sql
COMMIT;
```

**Reason:**

- Makes completed work permanent
- Releases locks immediately
- Limits rollback scope
- Improves PostgreSQL performance

If a crash happens later, previously committed batches remain safe.

**Decision:** Commit after every batch.

### 6. Exception Handling

Each batch is wrapped in an exception block.

If an error occurs:

- Current batch is rolled back
- Previous committed batches remain unchanged
- Error is logged
- Procedure exits safely

This prevents corruption and allows easy restart.

**Decision:** Use `EXCEPTION WHEN OTHERS` with `ROLLBACK`.

### 7. Using Primary Key Range Scanning

Records are processed using:

```sql
WHERE customer_id > start_id
  AND customer_id <= current_end
```

Since `customer_id` is the primary key, PostgreSQL can use the index efficiently. This is much faster than using `LIMIT` and `OFFSET`, which become slower for large tables.

**Decision:** Use indexed primary key ranges for scalability.

## Failure Recovery Example

Suppose processing succeeds up to customer ID `620000`.

The checkpoint table contains:

```text
last_processed_id = 620000
```

If the system crashes, the next run starts from:

```text
customer_id = 620001
```

No previous work is repeated.
