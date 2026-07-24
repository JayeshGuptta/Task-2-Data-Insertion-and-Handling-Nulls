# Task-2-Data-Insertion-and-Handling-Nulls
A practical SQL exercise demonstrating how to correctly **insert, update, delete, and handle NULL values** using a realistic `employees` table. Built and tested on [DB Fiddle](https://www.db-fiddle.com/).

## 📋 Table Overview

The `employees` table models a real company structure where some fields are optional by nature — not every employee has a manager, a phone number on file, or a sales commission.

```sql
CREATE TABLE employees (
    emp_id      INT PRIMARY KEY,
    emp_name    VARCHAR(50) NOT NULL,
    department  VARCHAR(30),
    salary      DECIMAL(10,2),
    manager_id  INT,
    phone       VARCHAR(15),
    commission  DECIMAL(10,2)
);
```

| Column | Nullable? | Why |
|---|---|---|
| `emp_id` | ❌ No (Primary Key) | Every row needs a unique identifier |
| `emp_name` | ❌ No | An employee record must have a name |
| `department` | ✅ Yes | May be unassigned temporarily |
| `salary` | ✅ Yes | May not be finalized yet |
| `manager_id` | ✅ Yes | The top-level employee (e.g. CEO) has no manager |
| `phone` | ✅ Yes | Contact info is optional |
| `commission` | ✅ Yes | Only applies to sales roles |

## 🛠️ What This Project Covers

1. Inserting records — both complete rows and rows with intentional/implicit NULLs
2. Handling NULL values efficiently — the core focus of this project
3. Updating records, including resolving NULLs into meaningful defaults
4. Deleting records safely using NULL-aware conditions

## 1️⃣ INSERT INTO — Creating Records

```sql
-- Full row insert
INSERT INTO employees (emp_id, emp_name, department, salary, manager_id, phone, commission)
VALUES (1, 'Alice Johnson', 'Sales', 55000.00, NULL, '9876543210', 5000.00);

-- CEO with no manager (explicit NULL)
INSERT INTO employees (emp_id, emp_name, department, salary, manager_id, phone, commission)
VALUES (2, 'Robert King', 'Executive', 120000.00, NULL, NULL, NULL);

-- Partial insert — omitted columns become NULL automatically
INSERT INTO employees (emp_id, emp_name, department, salary)
VALUES (3, 'Priya Sharma', 'IT', 62000.00);

INSERT INTO employees (emp_id, emp_name, department, salary, manager_id)
VALUES (4, 'David Lee', 'Sales', 48000.00, 1);

INSERT INTO employees (emp_id, emp_name, department, salary, manager_id, commission)
VALUES (5, 'Sara Khan', 'Sales', 51000.00, 1, 3000.00);
```

**Key takeaway:** any column left out of an `INSERT` statement is automatically set to `NULL` (unless a `DEFAULT` is defined) — this is how most real-world NULLs are created.

## 2️⃣ Handling NULL Values Efficiently

This is the most important part of the project. NULL means **"unknown / not applicable"**, not zero or empty string, and it behaves differently from every other value in SQL.

### ❌ The most common mistake
```sql
SELECT * FROM employees WHERE commission = NULL;  -- always returns 0 rows
```
NULL can never equal anything — not a value, not even another NULL.

### ✅ Correct way to check for NULL
```sql
SELECT * FROM employees WHERE commission IS NULL;
SELECT * FROM employees WHERE commission IS NOT NULL;
```

### ✅ Replace NULL with a default value — `COALESCE`
```sql
SELECT emp_name,
       COALESCE(commission, 0) AS commission,
       COALESCE(phone, 'Not Provided') AS phone
FROM employees;
```
`COALESCE(a, b, c, ...)` returns the first non-NULL value in the list — the standard, portable way (works in MySQL, PostgreSQL, SQL Server) to give NULLs a safe fallback value.

### ✅ Sorting NULLs predictably
```sql
-- PostgreSQL
SELECT * FROM employees ORDER BY commission NULLS LAST;
```
### 📌 Quick Reference

| Task | ❌ Wrong | ✅ Right |
|---|---|---|
| Find NULLs | `col = NULL` | `col IS NULL` |
| Exclude NULLs | `col != NULL` | `col IS NOT NULL` |
| Give a default value | ignoring it | `COALESCE(col, default)` |
| Math with a nullable column | `col1 + col2` | `col1 + COALESCE(col2, 0)` |
| Count non-null values | `COUNT(*)` | `COUNT(col)` |

## 3️⃣ UPDATE — Modifying Records & Resolving NULLs

```sql
-- Give Priya a manager
UPDATE employees
SET manager_id = 2
WHERE emp_id = 3;

-- Replace missing commission values with 0
UPDATE employees
SET commission = 0
WHERE commission IS NULL;
```

## 4️⃣ DELETE — Removing Records

```sql
-- Delete a specific employee
DELETE FROM employees
WHERE emp_id = 2;

## ▶️ How to Run This

1. Go to [DB Fiddle](https://www.db-fiddle.com/f/uMZUrVdYg1sQXZkaWdoneY/0#&togetherjs=dw9fzlG0Hu/)
2. Paste any `SELECT`, `UPDATE`, or `DELETE` statement from Sections 2–4 into the **Query** panel and click **Run**
3. Run a `SELECT * FROM employees;` between steps to see how the data changes

## ✅ Summary

| Concept | Demonstrated By |
| Inserting data (full & partial) | Section 1 |
| Detecting NULLs correctly | `IS NULL` / `IS NOT NULL` |
| Defaulting NULLs safely | `COALESCE()` |
| Avoiding NULL propagation in math | `COALESCE()` inside expressions |
| Updating records / resolving NULLs | Section 3 |
| Deleting records with NULL conditions | Section 4 |
