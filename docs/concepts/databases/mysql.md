# MySQL - Complete Guide

MySQL is the world's most popular open-source relational database management system (RDBMS). It's widely used for web applications and is a central component of the LAMP (Linux, Apache, MySQL, PHP/Python/Perl) stack.

## Installation

### Linux (Ubuntu/Debian)
```bash
# Update package index
sudo apt update

# Install MySQL Server
sudo apt install mysql-server

# Secure installation
sudo mysql_secure_installation

# Check status
sudo systemctl status mysql
```

### macOS
```bash
# Using Homebrew
brew install mysql

# Start MySQL
brew services start mysql

# Secure installation
mysql_secure_installation
```

### Windows
Download and install from [MySQL Official Website](https://dev.mysql.com/downloads/installer/)

## Basic Commands

### Connecting to MySQL

```bash
# Connect as root
mysql -u root -p

# Connect to specific database
mysql -u username -p database_name

# Connect to remote server
mysql -h hostname -u username -p database_name
```

### Database Operations

```sql
-- Show all databases
SHOW DATABASES;

-- Create database
CREATE DATABASE company;

-- Use database
USE company;

-- Delete database
DROP DATABASE company;

-- Show current database
SELECT DATABASE();
```

## Data Types

### Numeric Types

```sql
-- Integer types
TINYINT      -- 1 byte (-128 to 127)
SMALLINT     -- 2 bytes (-32,768 to 32,767)
MEDIUMINT    -- 3 bytes
INT          -- 4 bytes (-2,147,483,648 to 2,147,483,647)
BIGINT       -- 8 bytes

-- Decimal types
DECIMAL(10,2)  -- Fixed-point (10 digits, 2 after decimal)
FLOAT          -- Floating-point (4 bytes)
DOUBLE         -- Double precision (8 bytes)
```

### String Types

```sql
CHAR(50)       -- Fixed length (0-255)
VARCHAR(255)   -- Variable length (0-65,535)
TEXT           -- Long text (up to 65,535 characters)
MEDIUMTEXT     -- Medium text (up to 16,777,215 characters)
LONGTEXT       -- Very long text (up to 4,294,967,295 characters)
ENUM('M','F')  -- Enumeration
```

### Date and Time Types

```sql
DATE           -- YYYY-MM-DD
TIME           -- HH:MM:SS
DATETIME       -- YYYY-MM-DD HH:MM:SS
TIMESTAMP      -- Unix timestamp
YEAR           -- YYYY
```

## Creating Tables

### Basic Table Creation

```sql
CREATE TABLE employees (
    id INT PRIMARY KEY AUTO_INCREMENT,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    email VARCHAR(100) UNIQUE,
    phone VARCHAR(20),
    hire_date DATE,
    salary DECIMAL(10,2),
    department_id INT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
```

### Table with Foreign Keys

```sql
CREATE TABLE departments (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100) NOT NULL,
    location VARCHAR(100)
);

CREATE TABLE employees (
    id INT PRIMARY KEY AUTO_INCREMENT,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    email VARCHAR(100) UNIQUE,
    department_id INT,
    FOREIGN KEY (department_id) REFERENCES departments(id)
        ON DELETE SET NULL
        ON UPDATE CASCADE
);
```

### Table Management

```sql
-- Show all tables
SHOW TABLES;

-- Describe table structure
DESCRIBE employees;
-- or
SHOW COLUMNS FROM employees;

-- Show create table statement
SHOW CREATE TABLE employees;

-- Rename table
RENAME TABLE employees TO staff;

-- Drop table
DROP TABLE employees;

-- Truncate table (delete all data)
TRUNCATE TABLE employees;
```

## Modifying Tables

### Adding Columns

```sql
-- Add single column
ALTER TABLE employees
ADD COLUMN middle_name VARCHAR(50);

-- Add column with position
ALTER TABLE employees
ADD COLUMN age INT AFTER last_name;

-- Add column at beginning
ALTER TABLE employees
ADD COLUMN employee_code VARCHAR(20) FIRST;
```

### Modifying Columns

```sql
-- Change column type
ALTER TABLE employees
MODIFY COLUMN salary DECIMAL(12,2);

-- Rename column
ALTER TABLE employees
CHANGE COLUMN phone phone_number VARCHAR(20);

-- Drop column
ALTER TABLE employees
DROP COLUMN middle_name;
```

### Adding Constraints

```sql
-- Add primary key
ALTER TABLE employees
ADD PRIMARY KEY (id);

-- Add foreign key
ALTER TABLE employees
ADD CONSTRAINT fk_department
FOREIGN KEY (department_id) REFERENCES departments(id);

-- Add unique constraint
ALTER TABLE employees
ADD UNIQUE (email);

-- Add check constraint
ALTER TABLE employees
ADD CONSTRAINT chk_salary CHECK (salary > 0);

-- Add index
ALTER TABLE employees
ADD INDEX idx_last_name (last_name);
```

## CRUD Operations

### INSERT - Create Data

```sql
-- Insert single row
INSERT INTO employees (first_name, last_name, email, salary)
VALUES ('John', 'Doe', 'john.doe@example.com', 50000);

-- Insert multiple rows
INSERT INTO employees (first_name, last_name, email, salary)
VALUES 
    ('Jane', 'Smith', 'jane.smith@example.com', 55000),
    ('Bob', 'Johnson', 'bob.johnson@example.com', 60000),
    ('Alice', 'Williams', 'alice.williams@example.com', 52000);

-- Insert from another table
INSERT INTO employees_backup
SELECT * FROM employees WHERE department_id = 1;

-- Insert with ON DUPLICATE KEY UPDATE
INSERT INTO employees (id, first_name, last_name, salary)
VALUES (1, 'John', 'Doe', 50000)
ON DUPLICATE KEY UPDATE salary = 55000;
```

### SELECT - Read Data

```sql
-- Select all columns
SELECT * FROM employees;

-- Select specific columns
SELECT first_name, last_name, salary FROM employees;

-- Select with alias
SELECT 
    first_name AS 'First Name',
    last_name AS 'Last Name',
    salary AS 'Annual Salary'
FROM employees;

-- Select distinct values
SELECT DISTINCT department_id FROM employees;

-- Select with WHERE clause
SELECT * FROM employees WHERE salary > 50000;

-- Multiple conditions
SELECT * FROM employees 
WHERE salary > 50000 AND department_id = 1;

-- OR condition
SELECT * FROM employees 
WHERE department_id = 1 OR department_id = 2;

-- IN operator
SELECT * FROM employees 
WHERE department_id IN (1, 2, 3);

-- BETWEEN operator
SELECT * FROM employees 
WHERE salary BETWEEN 40000 AND 60000;

-- LIKE operator (pattern matching)
SELECT * FROM employees 
WHERE last_name LIKE 'S%';  -- Starts with S

SELECT * FROM employees 
WHERE email LIKE '%@example.com';  -- Ends with @example.com

-- IS NULL / IS NOT NULL
SELECT * FROM employees WHERE phone IS NULL;
SELECT * FROM employees WHERE phone IS NOT NULL;

-- ORDER BY
SELECT * FROM employees ORDER BY salary DESC;
SELECT * FROM employees ORDER BY last_name ASC, first_name ASC;

-- LIMIT
SELECT * FROM employees LIMIT 10;
SELECT * FROM employees LIMIT 10 OFFSET 20;  -- Skip 20, get 10
```

### UPDATE - Modify Data

```sql
-- Update single row
UPDATE employees
SET salary = 55000
WHERE id = 1;

-- Update multiple columns
UPDATE employees
SET 
    salary = 60000,
    department_id = 2
WHERE id = 1;

-- Update with calculation
UPDATE employees
SET salary = salary * 1.10
WHERE department_id = 1;

-- Update all rows (be careful!)
UPDATE employees
SET updated_at = NOW();
```

### DELETE - Remove Data

```sql
-- Delete specific rows
DELETE FROM employees WHERE id = 1;

-- Delete with condition
DELETE FROM employees WHERE salary < 30000;

-- Delete all rows (be very careful!)
DELETE FROM employees;
```

## Joins

### INNER JOIN

```sql
-- Get employees with their department names
SELECT 
    e.first_name,
    e.last_name,
    d.name AS department
FROM employees e
INNER JOIN departments d ON e.department_id = d.id;
```

### LEFT JOIN

```sql
-- Get all employees, including those without departments
SELECT 
    e.first_name,
    e.last_name,
    d.name AS department
FROM employees e
LEFT JOIN departments d ON e.department_id = d.id;
```

### RIGHT JOIN

```sql
-- Get all departments, including those without employees
SELECT 
    e.first_name,
    e.last_name,
    d.name AS department
FROM employees e
RIGHT JOIN departments d ON e.department_id = d.id;
```

### CROSS JOIN

```sql
-- Cartesian product of two tables
SELECT 
    e.first_name,
    d.name
FROM employees e
CROSS JOIN departments d;
```

### Self Join

```sql
-- Find employees with the same salary
SELECT 
    e1.first_name AS employee1,
    e2.first_name AS employee2,
    e1.salary
FROM employees e1
INNER JOIN employees e2 ON e1.salary = e2.salary AND e1.id != e2.id;
```

## Aggregate Functions

```sql
-- COUNT
SELECT COUNT(*) FROM employees;
SELECT COUNT(DISTINCT department_id) FROM employees;

-- SUM
SELECT SUM(salary) AS total_payroll FROM employees;

-- AVG
SELECT AVG(salary) AS average_salary FROM employees;

-- MIN and MAX
SELECT MIN(salary) AS lowest_salary FROM employees;
SELECT MAX(salary) AS highest_salary FROM employees;

-- GROUP BY
SELECT 
    department_id,
    COUNT(*) AS employee_count,
    AVG(salary) AS avg_salary
FROM employees
GROUP BY department_id;

-- HAVING (filter groups)
SELECT 
    department_id,
    COUNT(*) AS employee_count
FROM employees
GROUP BY department_id
HAVING COUNT(*) > 5;

-- GROUP BY with JOIN
SELECT 
    d.name AS department,
    COUNT(e.id) AS employee_count,
    AVG(e.salary) AS avg_salary
FROM departments d
LEFT JOIN employees e ON d.id = e.department_id
GROUP BY d.id, d.name;
```

## Subqueries

```sql
-- Subquery in WHERE clause
SELECT first_name, last_name, salary
FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees);

-- Subquery with IN
SELECT first_name, last_name
FROM employees
WHERE department_id IN (
    SELECT id FROM departments WHERE location = 'New York'
);

-- Subquery in FROM clause
SELECT dept_name, avg_salary
FROM (
    SELECT 
        d.name AS dept_name,
        AVG(e.salary) AS avg_salary
    FROM departments d
    JOIN employees e ON d.id = e.department_id
    GROUP BY d.id, d.name
) AS dept_stats
WHERE avg_salary > 50000;

-- Correlated subquery
SELECT e1.first_name, e1.last_name, e1.salary
FROM employees e1
WHERE salary > (
    SELECT AVG(salary)
    FROM employees e2
    WHERE e2.department_id = e1.department_id
);
```

## Indexes

### Creating Indexes

```sql
-- Single column index
CREATE INDEX idx_last_name ON employees(last_name);

-- Composite index
CREATE INDEX idx_name ON employees(last_name, first_name);

-- Unique index
CREATE UNIQUE INDEX idx_email ON employees(email);

-- Full-text index
CREATE FULLTEXT INDEX idx_description ON products(description);
```

### Managing Indexes

```sql
-- Show indexes
SHOW INDEX FROM employees;

-- Drop index
DROP INDEX idx_last_name ON employees;

-- Analyze index usage
EXPLAIN SELECT * FROM employees WHERE last_name = 'Smith';
```

## Views

```sql
-- Create view
CREATE VIEW employee_details AS
SELECT 
    e.id,
    e.first_name,
    e.last_name,
    e.email,
    e.salary,
    d.name AS department
FROM employees e
LEFT JOIN departments d ON e.department_id = d.id;

-- Use view
SELECT * FROM employee_details WHERE salary > 50000;

-- Update view
CREATE OR REPLACE VIEW employee_details AS
SELECT 
    e.id,
    CONCAT(e.first_name, ' ', e.last_name) AS full_name,
    e.email,
    e.salary,
    d.name AS department
FROM employees e
LEFT JOIN departments d ON e.department_id = d.id;

-- Drop view
DROP VIEW employee_details;
```

## Stored Procedures

```sql
-- Create stored procedure
DELIMITER //

CREATE PROCEDURE GetEmployeesByDepartment(IN dept_id INT)
BEGIN
    SELECT * FROM employees WHERE department_id = dept_id;
END //

DELIMITER ;

-- Call stored procedure
CALL GetEmployeesByDepartment(1);

-- Procedure with OUT parameter
DELIMITER //

CREATE PROCEDURE GetEmployeeCount(IN dept_id INT, OUT emp_count INT)
BEGIN
    SELECT COUNT(*) INTO emp_count
    FROM employees
    WHERE department_id = dept_id;
END //

DELIMITER ;

-- Call with OUT parameter
CALL GetEmployeeCount(1, @count);
SELECT @count;

-- Drop procedure
DROP PROCEDURE GetEmployeesByDepartment;
```

## Triggers

```sql
-- Create trigger
DELIMITER //

CREATE TRIGGER before_employee_insert
BEFORE INSERT ON employees
FOR EACH ROW
BEGIN
    SET NEW.created_at = NOW();
    SET NEW.email = LOWER(NEW.email);
END //

DELIMITER ;

-- Audit trigger
CREATE TABLE employee_audit (
    id INT PRIMARY KEY AUTO_INCREMENT,
    employee_id INT,
    action VARCHAR(50),
    changed_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

DELIMITER //

CREATE TRIGGER after_employee_update
AFTER UPDATE ON employees
FOR EACH ROW
BEGIN
    INSERT INTO employee_audit (employee_id, action)
    VALUES (NEW.id, 'UPDATE');
END //

DELIMITER ;

-- Show triggers
SHOW TRIGGERS;

-- Drop trigger
DROP TRIGGER before_employee_insert;
```

## Transactions

```sql
-- Start transaction
START TRANSACTION;

-- Perform operations
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;

-- Commit transaction
COMMIT;

-- Or rollback if error
ROLLBACK;

-- Transaction with savepoint
START TRANSACTION;

UPDATE accounts SET balance = balance - 100 WHERE id = 1;
SAVEPOINT sp1;

UPDATE accounts SET balance = balance + 100 WHERE id = 2;
SAVEPOINT sp2;

-- Rollback to savepoint
ROLLBACK TO sp1;

COMMIT;
```

## User Management

```sql
-- Create user
CREATE USER 'john'@'localhost' IDENTIFIED BY 'password123';

-- Grant privileges
GRANT ALL PRIVILEGES ON company.* TO 'john'@'localhost';
GRANT SELECT, INSERT ON company.employees TO 'john'@'localhost';

-- Show grants
SHOW GRANTS FOR 'john'@'localhost';

-- Revoke privileges
REVOKE INSERT ON company.employees FROM 'john'@'localhost';

-- Change password
ALTER USER 'john'@'localhost' IDENTIFIED BY 'newpassword123';

-- Drop user
DROP USER 'john'@'localhost';

-- Flush privileges
FLUSH PRIVILEGES;
```

## Backup and Restore

### Backup

```bash
# Backup single database
mysqldump -u root -p company > company_backup.sql

# Backup all databases
mysqldump -u root -p --all-databases > all_databases_backup.sql

# Backup specific tables
mysqldump -u root -p company employees departments > tables_backup.sql

# Backup with compression
mysqldump -u root -p company | gzip > company_backup.sql.gz

# Backup structure only (no data)
mysqldump -u root -p --no-data company > company_structure.sql
```

### Restore

```bash
# Restore database
mysql -u root -p company < company_backup.sql

# Restore compressed backup
gunzip < company_backup.sql.gz | mysql -u root -p company

# Restore all databases
mysql -u root -p < all_databases_backup.sql
```

## Performance Optimization

### Query Optimization

```sql
-- Use EXPLAIN to analyze queries
EXPLAIN SELECT * FROM employees WHERE last_name = 'Smith';

-- Use indexes effectively
CREATE INDEX idx_last_name ON employees(last_name);

-- Avoid SELECT *
SELECT id, first_name, last_name FROM employees;

-- Use LIMIT for large result sets
SELECT * FROM employees LIMIT 100;

-- Use JOIN instead of subqueries when possible
-- Bad
SELECT * FROM employees 
WHERE department_id IN (SELECT id FROM departments WHERE location = 'NY');

-- Better
SELECT e.* FROM employees e
INNER JOIN departments d ON e.department_id = d.id
WHERE d.location = 'NY';
```

### Configuration Tuning

```sql
-- Show variables
SHOW VARIABLES LIKE 'max_connections';

-- Set variables
SET GLOBAL max_connections = 200;

-- Important variables to tune
-- innodb_buffer_pool_size (50-80% of RAM)
-- max_connections
-- query_cache_size
-- tmp_table_size
-- max_heap_table_size
```

## Common Functions

### String Functions

```sql
-- CONCAT
SELECT CONCAT(first_name, ' ', last_name) AS full_name FROM employees;

-- UPPER / LOWER
SELECT UPPER(first_name), LOWER(last_name) FROM employees;

-- SUBSTRING
SELECT SUBSTRING(email, 1, 5) FROM employees;

-- LENGTH
SELECT first_name, LENGTH(first_name) AS name_length FROM employees;

-- TRIM
SELECT TRIM('  hello  ');

-- REPLACE
SELECT REPLACE(email, '@example.com', '@company.com') FROM employees;
```

### Date Functions

```sql
-- NOW / CURDATE / CURTIME
SELECT NOW(), CURDATE(), CURTIME();

-- DATE_FORMAT
SELECT DATE_FORMAT(hire_date, '%Y-%m-%d') FROM employees;

-- DATEDIFF
SELECT DATEDIFF(NOW(), hire_date) AS days_employed FROM employees;

-- DATE_ADD / DATE_SUB
SELECT DATE_ADD(hire_date, INTERVAL 1 YEAR) FROM employees;

-- YEAR / MONTH / DAY
SELECT YEAR(hire_date), MONTH(hire_date), DAY(hire_date) FROM employees;
```

### Numeric Functions

```sql
-- ROUND
SELECT ROUND(salary, 2) FROM employees;

-- CEIL / FLOOR
SELECT CEIL(salary / 12), FLOOR(salary / 12) FROM employees;

-- ABS
SELECT ABS(-100);

-- RAND
SELECT * FROM employees ORDER BY RAND() LIMIT 5;
```

## Best Practices

1. **Use Appropriate Data Types**: Choose the smallest data type that fits your needs
2. **Normalize Your Database**: Reduce redundancy through normalization
3. **Use Indexes Wisely**: Index columns used in WHERE, JOIN, and ORDER BY
4. **Avoid SELECT ***: Only select columns you need
5. **Use Transactions**: For operations that must succeed or fail together
6. **Regular Backups**: Automate database backups
7. **Parameterized Queries**: Prevent SQL injection
8. **Monitor Performance**: Use EXPLAIN and slow query log
9. **Keep MySQL Updated**: Apply security patches
10. **Use Connection Pooling**: Reuse database connections

## Security Best Practices

```sql
-- Use strong passwords
ALTER USER 'root'@'localhost' IDENTIFIED BY 'StrongP@ssw0rd!';

-- Limit user privileges
GRANT SELECT, INSERT, UPDATE ON company.* TO 'app_user'@'localhost';

-- Remove anonymous users
DELETE FROM mysql.user WHERE User='';

-- Disable remote root login
DELETE FROM mysql.user WHERE User='root' AND Host NOT IN ('localhost', '127.0.0.1');

-- Use SSL connections
-- In my.cnf:
-- [mysqld]
-- require_secure_transport=ON
```

## Troubleshooting

```sql
-- Check MySQL status
SHOW STATUS;

-- Show running processes
SHOW PROCESSLIST;

-- Kill long-running query
KILL QUERY process_id;

-- Check table for errors
CHECK TABLE employees;

-- Repair table
REPAIR TABLE employees;

-- Optimize table
OPTIMIZE TABLE employees;

-- Show errors
SHOW ERRORS;

-- Show warnings
SHOW WARNINGS;
```

## Resources

- [MySQL Official Documentation](https://dev.mysql.com/doc/)
- [MySQL Tutorial](https://www.mysqltutorial.org/)
- [MySQL Performance Blog](https://www.percona.com/blog/)

---

- Go back to [Databases](./index.md)
- Return to [Home](../../index.md)
