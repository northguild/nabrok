# PostgreSQL Cheatsheet

A crash course in PostgreSQL — what it is, how it works, and the commands you'll actually use.

## What is PostgreSQL?

PostgreSQL (or "Postgres") is a **relational database** — one of the most advanced and standards-compliant open-source databases in the world. It stores data in **tables** with **rows** and **columns**, just like a spreadsheet, but with superpowers.

### Why Relational?

"Relational" means data is organized into tables that can be **related** to each other through keys. For example:

```
Users Table          Orders Table
┌────┬──────────┐    ┌──────┬───────┬────────┐
│ id │ name     │    │ id   │ user  │ amount │
├────┼──────────┤    ├──────┼───────┼────────┤
│ 1  │ Alice    │◄───│ 101  │ 1     │ 99.99  │
│ 2  │ Bob      │    │ 102  │ 1     │ 49.50  │
│ 3  │ Charlie  │    │ 103  │ 2     │ 25.00  │
└────┴──────────┘    └──────┴───────┴────────┘
```

The `user` column in Orders **references** the `id` in Users. This is a **foreign key**.

### SQL: The Language

SQL (Structured Query Language) is how you talk to any relational database. PostgreSQL uses a dialect called **PostgreSQL SQL**. It has 4 main operations:

| Operation | SQL Keyword | What it does |
|-----------|-------------|--------------|
| **Create** | `INSERT` | Add new rows |
| **Read** | `SELECT` | Query existing rows |
| **Update** | `UPDATE` | Modify existing rows |
| **Delete** | `DELETE` | Remove rows |

Think of these as **CRUD** — Create, Read, Update, Delete. Every app does these 4 things with data.

## Getting Started with `psql`

`psql` is the command-line tool for talking to PostgreSQL. Connect like this:

```bash
psql -h localhost -p 5432 -U postgres -d appdb
```

Or from inside Docker:

```bash
docker exec -it postgres psql -U postgres -d appdb
```

## Crash Course: Creating Data

### 1. Create a Table

A table is like a spreadsheet — it defines the **schema** (structure) of your data.

```sql
-- Create a users table
CREATE TABLE users (
    id          SERIAL PRIMARY KEY,     -- Auto-incrementing number (1, 2, 3...)
    name        TEXT NOT NULL,           -- Text, cannot be empty
    email       TEXT UNIQUE NOT NULL,    -- Text, must be different for each row
    age         INTEGER,                 -- Whole number, can be null
    created_at  TIMESTAMP DEFAULT NOW()  -- Date/time, defaults to now
);

-- Create an orders table with a foreign key
CREATE TABLE orders (
    id          SERIAL PRIMARY KEY,
    user_id     INTEGER REFERENCES users(id),  -- Links to users.id
    amount      NUMERIC(10, 2) NOT NULL,        -- Decimal number (e.g., 99.99)
    status      TEXT DEFAULT 'pending',
    created_at  TIMESTAMP DEFAULT NOW()
);
```

### 2. Insert Data

```sql
-- Insert one row
INSERT INTO users (name, email, age)
VALUES ('Alice', 'alice@example.com', 30);

-- Insert multiple rows at once
INSERT INTO users (name, email, age)
VALUES
    ('Bob', 'bob@example.com', 25),
    ('Charlie', 'charlie@example.com', 35);

-- Insert an order linked to Alice (user_id = 1)
INSERT INTO orders (user_id, amount, status)
VALUES (1, 99.99, 'completed');
```

## Crash Course: Reading Data

### SELECT Basics

```sql
-- Get ALL columns for ALL users
SELECT * FROM users;

-- Get specific columns only
SELECT name, email FROM users;

-- Get users with a condition (WHERE)
SELECT name, email FROM users WHERE age > 25;

-- Multiple conditions (AND, OR)
SELECT * FROM users
WHERE age >= 25 AND age <= 35;

-- Order results (ORDER BY)
SELECT name, age FROM users ORDER BY age DESC;   -- oldest first
SELECT name, age FROM users ORDER BY age ASC;     -- youngest first

-- Limit results (LIMIT)
SELECT * FROM users ORDER BY created_at DESC LIMIT 5;  -- 5 most recent

-- Check if a value exists (EXISTS)
SELECT EXISTS (SELECT 1 FROM users WHERE email = 'alice@example.com');
```

### JOINs: Combining Tables

JOINs are how you get data from multiple tables together. This is the **most important concept** in relational databases.

```sql
-- INNER JOIN: Only rows that match in BOTH tables
SELECT users.name, orders.amount, orders.status
FROM users
JOIN orders ON users.id = orders.user_id;

-- Result:
-- name   | amount | status
-- Alice  | 99.99  | completed

-- LEFT JOIN: ALL users, even those without orders
SELECT users.name, orders.amount
FROM users
LEFT JOIN orders ON users.id = orders.user_id;

-- Result (Bob and Charlie appear with NULL because they have no orders):
-- name    | amount
-- Alice   | 99.99
-- Bob     | NULL
-- Charlie | NULL
```

### Aggregation: Summarizing Data

```sql
-- Count rows
SELECT COUNT(*) FROM users;                          -- Total users
SELECT COUNT(*) FROM orders WHERE status = 'completed';  -- Completed orders

-- Sum a column
SELECT SUM(amount) FROM orders;                      -- Total revenue
SELECT SUM(amount) FROM orders WHERE status = 'completed';  -- Completed revenue

-- Average
SELECT AVG(amount) FROM orders;                      -- Average order value

-- Min / Max
SELECT MIN(amount), MAX(amount) FROM orders;         -- Smallest and largest order

-- Group by (the most powerful aggregation)
SELECT users.name, COUNT(*) as order_count, SUM(amount) as total_spent
FROM users
JOIN orders ON users.id = orders.user_id
GROUP BY users.name
ORDER BY total_spent DESC;

-- Result:
-- name    | order_count | total_spent
-- Alice   | 3           | 299.97
-- Bob     | 1           | 49.50
```

### Filtering Groups (HAVING)

```sql
-- GROUP BY filters rows, HAVING filters groups
SELECT users.name, COUNT(*) as order_count
FROM users
JOIN orders ON users.id = orders.user_id
GROUP BY users.name
HAVING COUNT(*) > 1;  -- Only users with more than 1 order
```

## Crash Course: Modifying Data

### UPDATE

```sql
-- Update one row
UPDATE users SET age = 31 WHERE id = 1;

-- Update multiple columns
UPDATE users SET name = 'Alice Smith', age = 31 WHERE id = 1;

-- Update based on another table
UPDATE orders SET status = 'shipped'
WHERE id IN (SELECT order_id FROM shipments WHERE tracking IS NOT NULL);
```

### DELETE

```sql
-- Delete specific rows
DELETE FROM orders WHERE id = 101;

-- Delete with a condition
DELETE FROM users WHERE age < 18;

-- DANGER: Delete ALL rows (no WHERE clause!)
DELETE FROM users;  -- ⚠️ This removes everything!

-- Safer: check first
SELECT * FROM users WHERE age < 18;  -- Review what will be deleted
DELETE FROM users WHERE age < 18;    -- Then delete
```

## PostgreSQL Data Types Quick Reference

| Type | Use For | Example |
|------|---------|---------|
| `SERIAL` | Auto-incrementing ID | `1, 2, 3...` |
| `INTEGER` | Whole numbers | `42, -7, 1000` |
| `NUMERIC(p,s)` | Exact decimals | `NUMERIC(10,2)` → `99.99` |
| `TEXT` | Unlimited text | `'Hello world'` |
| `VARCHAR(n)` | Text with max length | `VARCHAR(255)` |
| `BOOLEAN` | True/false | `true, false` |
| `DATE` | Date only | `'2026-09-16'` |
| `TIMESTAMP` | Date + time | `'2026-09-16 14:30:00'` |
| `JSONB` | Binary JSON (indexed!) | `'{"key": "value"}'` |
| `UUID` | Unique identifier | `'550e8400-e29b-41d4-a716-446655440000'` |

## Useful psql Commands (Meta-Commands)

These start with `\` and are interpreted by `psql`, not PostgreSQL:

```sql
\l                          -- List all databases
\c <database>               -- Connect to a different database
\dt                         -- List tables in current database
\d <table_name>             -- Describe a table (columns, types, keys)
\d                          -- Describe all tables
\du                         -- List users (roles)
\di                         -- List indexes
\dn                         -- List schemas
\q                          -- Quit psql
\?                          -- Show all psql commands
```

## pgAdmin Tips

- **Query Tool**: Click "Query Tool" on any table to run SQL queries visually
- **Browse Data**: Right-click a table → "View/Edit Data" → "All Rows"
- **Create Table**: Right-click Tables → "Create" → "Table"
- **Import/Export**: Right-click a table → "Import/Export Data"
- **Explain Plan**: Add `EXPLAIN ANALYZE` before any SELECT to see performance details

## Common Patterns

### Pagination

```sql
-- Page 2, 10 items per page
SELECT * FROM users ORDER BY id LIMIT 10 OFFSET 10;
-- OFFSET = (page - 1) * limit
```

### Soft Delete (mark instead of removing)

```sql
ALTER TABLE users ADD COLUMN deleted_at TIMESTAMP;

-- "Delete" a user
UPDATE users SET deleted_at = NOW() WHERE id = 1;

-- Query only active users
SELECT * FROM users WHERE deleted_at IS NULL;
```

### Upsert (insert or update)

```sql
-- Insert, but if the email already exists, update the age
INSERT INTO users (name, email, age)
VALUES ('Alice', 'alice@example.com', 31)
ON CONFLICT (email) DO UPDATE SET age = EXCLUDED.age;
```

### Date/Time Operations

```sql
-- Current date/time
SELECT NOW();                          -- Current timestamp
SELECT CURRENT_DATE;                   -- Today's date
SELECT CURRENT_TIME;                   -- Current time

-- Date arithmetic
SELECT NOW() + INTERVAL '7 days';      -- 7 days from now
SELECT NOW() - INTERVAL '1 month';     -- 1 month ago

-- Extract parts
SELECT EXTRACT(YEAR FROM created_at) FROM orders;   -- Year
SELECT EXTRACT(MONTH FROM created_at) FROM orders;  -- Month
SELECT EXTRACT(HOUR FROM created_at) FROM orders;   -- Hour

-- Format dates
SELECT TO_CHAR(created_at, 'YYYY-MM-DD') FROM orders;
SELECT TO_CHAR(created_at, 'Mon DD, YYYY') FROM orders;
```

## Next Steps

- **Indexes**: Speed up queries (covered in the next section)
- **Transactions**: Group operations atomically
- **Views**: Saved queries you can treat like tables
- **Functions/Procedures**: Reusable SQL logic
