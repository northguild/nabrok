# MySQL Cheatsheet

A crash course in MySQL — what it is, how it works, and the commands you'll actually use.

## What is MySQL?

MySQL is one of the most popular **relational databases** in the world. It was created in 1995 and is now owned by Oracle. It powers WordPress, Facebook, YouTube, and countless other applications.

### How Does It Compare to PostgreSQL?

Both are relational databases using SQL, but they have different philosophies:

| Aspect | MySQL | PostgreSQL |
|--------|-------|------------|
| **Philosophy** | Fast, simple, widely used | Advanced, standards-compliant |
| **JSON support** | Good (MySQL 5.7+) | Excellent (JSONB, indexed) |
| **Full-text search** | Built-in | Available but less mature |
| **Complex queries** | Good | Better (CTEs, window functions) |
| **Extensibility** | Plugins | Custom types, functions, languages |
| **Default engine** | InnoDB | N/A (single engine) |

For most web applications, either works great. MySQL is slightly more common with PHP/LAMP stacks. PostgreSQL is more common with Node.js/Python apps.

### SQL: The Language

MySQL uses **SQL** (Structured Query Language), the same language as PostgreSQL. The basic operations are identical:

| Operation | SQL Keyword | What it does |
|-----------|-------------|--------------|
| **Create** | `INSERT` | Add new rows |
| **Read** | `SELECT` | Query existing rows |
| **Update** | `UPDATE` | Modify existing rows |
| **Delete** | `DELETE` | Remove rows |

## Getting Started with MySQL

### Connect via Command Line

```bash
mysql -u root -p appdb
# You'll be prompted for the password
```

Or from inside Docker:

```bash
docker exec -it mysql mysql -u root -p appdb
```

### phpMyAdmin Tips

- **Browse**: Click a database → click a table to see all rows
- **SQL Tab**: Run custom queries in the query editor
- **Import/Export**: Click "Import" or "Export" on any table
- **Structure Tab**: See columns, indexes, foreign keys
- **Operations Tab**: Change table settings, optimize, repair

## Crash Course: Creating Data

### 1. Create a Table

```sql
-- Create a users table
CREATE TABLE users (
    id          INT AUTO_INCREMENT PRIMARY KEY,  -- Auto-incrementing ID
    name        VARCHAR(255) NOT NULL,            -- Text up to 255 chars
    email       VARCHAR(255) UNIQUE NOT NULL,     -- Unique email
    age         INT,                              -- Whole number
    created_at  TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at  TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);

-- Create an orders table with foreign key
CREATE TABLE orders (
    id          INT AUTO_INCREMENT PRIMARY KEY,
    user_id     INT NOT NULL,
    amount      DECIMAL(10, 2) NOT NULL,          -- Decimal: 99.99
    status      ENUM('pending', 'completed', 'shipped', 'cancelled') DEFAULT 'pending',
    created_at  TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);
```

### 2. Insert Data

```sql
-- Insert one row
INSERT INTO users (name, email, age)
VALUES ('Alice', 'alice@example.com', 30);

-- Insert multiple rows
INSERT INTO users (name, email, age)
VALUES
    ('Bob', 'bob@example.com', 25),
    ('Charlie', 'charlie@example.com', 35);

-- Insert an order
INSERT INTO orders (user_id, amount, status)
VALUES (1, 99.99, 'completed');
```

## Crash Course: Reading Data

### SELECT Basics

```sql
-- Get all columns
SELECT * FROM users;

-- Get specific columns
SELECT name, email FROM users;

-- Filter with WHERE
SELECT name, email FROM users WHERE age > 25;

-- Multiple conditions
SELECT * FROM users WHERE age >= 25 AND age <= 35;

-- ORDER BY (DESC = descending, ASC = ascending)
SELECT name, age FROM users ORDER BY age DESC;

-- LIMIT results
SELECT * FROM users ORDER BY created_at DESC LIMIT 5;

-- LIKE for pattern matching
SELECT * FROM users WHERE name LIKE 'A%';    -- Starts with A
SELECT * FROM users WHERE email LIKE '%@gmail.com';  -- Gmail users only
```

### JOINs: Combining Tables

```sql
-- INNER JOIN: Only matching rows
SELECT users.name, orders.amount, orders.status
FROM users
JOIN orders ON users.id = orders.user_id;

-- LEFT JOIN: All users, even without orders
SELECT users.name, orders.amount
FROM users
LEFT JOIN orders ON users.id = orders.user_id;
```

### Aggregation

```sql
-- Count
SELECT COUNT(*) FROM users;

-- Sum
SELECT SUM(amount) FROM orders;

-- Average
SELECT AVG(amount) FROM orders;

-- Min / Max
SELECT MIN(amount), MAX(amount) FROM orders;

-- Group by (powerful!)
SELECT users.name, COUNT(*) as order_count, SUM(amount) as total_spent
FROM users
JOIN orders ON users.id = orders.user_id
GROUP BY users.name
ORDER BY total_spent DESC;
```

## MySQL-Specific Features

### AUTO_INCREMENT

Automatically generates unique IDs:

```sql
-- No need to specify id — MySQL handles it
INSERT INTO users (name, email) VALUES ('Dave', 'dave@example.com');
-- id will be 4 (auto-assigned)
```

### ENUM

Restrict a column to specific values:

```sql
CREATE TABLE tasks (
    id INT AUTO_INCREMENT PRIMARY KEY,
    status ENUM('todo', 'in_progress', 'done', 'cancelled') DEFAULT 'todo'
);

-- Only these values are allowed
INSERT INTO tasks (status) VALUES ('in_progress');  -- ✅ OK
INSERT INTO tasks (status) VALUES ('urgent');        -- ❌ Error!
```

### TIMESTAMP with Auto-Update

```sql
CREATE TABLE logs (
    id INT AUTO_INCREMENT PRIMARY KEY,
    message TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
-- updated_at automatically updates whenever the row changes
```

### DECIMAL for Money

Always use `DECIMAL` for currency, never `FLOAT` or `DOUBLE`:

```sql
CREATE TABLE products (
    id INT AUTO_INCREMENT PRIMARY KEY,
    price DECIMAL(10, 2) NOT NULL  -- Up to 99999999.99
);

INSERT INTO products (price) VALUES (99.99);  -- Exact!
```

## Useful MySQL Commands

```sql
-- List all databases
SHOW DATABASES;

-- Use a database
USE appdb;

-- List tables
SHOW TABLES;

-- Describe a table
DESCRIBE users;          -- or \d users

-- Show table structure in detail
SHOW CREATE TABLE users;

-- Show indexes
SHOW INDEX FROM users;

-- Show running processes
SHOW PROCESSLIST;

-- Kill a slow query
KILL <process_id>;

-- Check server version
SELECT VERSION();
```

## Common Patterns

### Pagination

```sql
-- Page 2, 10 items per page
SELECT * FROM users ORDER BY id LIMIT 10 OFFSET 10;
```

### Soft Delete

```sql
ALTER TABLE users ADD COLUMN deleted_at TIMESTAMP NULL;

UPDATE users SET deleted_at = NOW() WHERE id = 1;
SELECT * FROM users WHERE deleted_at IS NULL;
```

### Upsert (Insert or Update)

```sql
INSERT INTO users (name, email, age)
VALUES ('Alice', 'alice@example.com', 31)
ON DUPLICATE KEY UPDATE age = VALUES(age);
```

### Date/Time Operations

```sql
-- Current date/time
SELECT NOW();
SELECT CURDATE();

-- Date arithmetic
SELECT NOW() + INTERVAL 7 DAY;       -- 7 days from now
SELECT NOW() - INTERVAL 1 MONTH;     -- 1 month ago

-- Extract parts
SELECT YEAR(created_at) FROM orders;
SELECT MONTH(created_at) FROM orders;
SELECT DAY(created_at) FROM orders;

-- Format dates
SELECT DATE_FORMAT(created_at, '%Y-%m-%d') FROM orders;
SELECT DATE_FORMAT(created_at, '%b %d, %Y') FROM orders;
```

## MySQL Data Types Quick Reference

| Type | Use For | Example |
|------|---------|---------|
| `INT` | Whole numbers | `42, -7, 1000` |
| `BIGINT` | Large whole numbers | `999999999999` |
| `DECIMAL(p,s)` | Exact decimals (money!) | `DECIMAL(10,2)` → `99.99` |
| `VARCHAR(n)` | Text up to n chars | `VARCHAR(255)` |
| `TEXT` | Unlimited text | `'Long article...'` |
| `BOOLEAN` | True/false | `TRUE, FALSE` |
| `DATE` | Date only | `'2026-09-16'` |
| `DATETIME` | Date + time | `'2026-09-16 14:30:00'` |
| `TIMESTAMP` | Date + time (auto-updates) | Auto-managed |
| `JSON` | JSON data | `'{"key": "value"}'` |
| `ENUM(...)` | Fixed set of values | `ENUM('a','b','c')` |

## phpMyAdmin Tips

- **Quick Actions**: Hover over a table to see quick actions (Browse, Structure, Search, Insert, Export, Drop)
- **Search**: Click "Search" on a table to filter rows visually
- **Insert**: Click "Insert" to add rows through a form
- **Export**: Click "Export" to download data as SQL, CSV, or Excel
- **Import**: Click "Import" to upload SQL files or CSV
- **SQL Tab**: Write and run custom queries
- **Operations**: Change collation, engine, or table options

## Next Steps

- **Indexes**: Speed up queries (CREATE INDEX)
- **Transactions**: Group operations atomically (START TRANSACTION)
- **Stored Procedures**: Reusable SQL logic
- **Views**: Saved queries treated like tables
