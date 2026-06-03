## These are the basics stuff for database that everyone should know about

## DDL Commands: Review and Clarifications

### 1. `CREATE` 
- Used to create databases, tables.
- It's used to establish the structure of any new database object (tables, databases, views, indexes, stored procedures, etc.).
- **Example:** `CREATE TABLE users (id INT, name VARCHAR(100));`

### 2. `ALTER` (Needs minor refinement)
- Used to modify columns, consisting of `modify`, `rename`, and `drop`.
    - **`ALTER TABLE`** is the main command. The sub-clauses (`MODIFY`, `RENAME`, `DROP`) define the action being taken on the columns.
	
    - **`MODIFY`**: Changes column **data type, size, or constraints** (Used in MySQL/Oracle).
	
    - **`RENAME`**: **Renames tables** (`ALTER TABLE old_name RENAME TO new_name;`) and is used to **rename columns** (`ALTER TABLE table_name RENAME COLUMN old_name TO new_name;`).
	
    - **`DROP`**: Correct. Used as **`ALTER TABLE table_name DROP COLUMN column_name;`** to delete a column.

### 3. `DROP` 
- Used to delete databases, tables as well as columns with combination to alter.
- **Clarification:** The key is that `DROP` is used on its own for **major objects** (Database, Table, View).
    - `DROP DATABASE database_name;`
    - `DROP TABLE table_name;`
    - To drop a **column**, the **`DROP`** clause must be nested inside **`ALTER TABLE`**. You cannot run `DROP COLUMN name;` by itself.

### 4. `TRUNCATE` 
- Used to delete all data from the table without deleting table definitions.
- It's a very fast way to empty a table, and because it deals with the data set as a whole (not row-by-row), it is classified as DDL and is typically **non-transactional** (auto-committed).
- **Example:** `TRUNCATE TABLE AuditLog;`

---

## Most Useful and Often Forgotten SQL Filters

## 1. `BETWEEN`
- **What it does:** Filters results by a range, inclusive of the start and end values.
- **Why it's useful:** Great for numeric or date ranges, and often overlooked in favour of `AND` conditions.

**Example:**
```sql
SELECT * 
FROM employees
WHERE salary BETWEEN 40000 AND 60000;
```

## 2. `IN` (vs multiple `OR` statements)
- **What it does:** Filters rows that match any value in a list.
- **Why it's useful:** Cleaner and easier to read when you need to check a column against multiple values.

**Example:**
```sql
SELECT *  FROM employees WHERE department_id IN (1, 2, 3);`
```

## 3. `IS NULL` / `IS NOT NULL`
- **What it does:** Filters for `NULL` values or non-`NULL` values.
- **Why it's useful:** Essential for handling missing or incomplete data, but often forgotten.

**Example (NULL):**
```sql
SELECT *  FROM employees WHERE manager_id IS NULL;
```

**Example (NOT NULL):**

```sql
SELECT *  FROM employees WHERE manager_id IS NOT NULL;`
```


## 4. `LIKE` (for pattern matching)

- **What it does:** Filters results based on pattern matching with wildcards.
- **Why it's useful:** Useful for searching strings or partial matches.

**Example (starts with "A"):**

```sql
SELECT *  FROM employees WHERE name LIKE 'A%';`
```

**Other wildcards:**

- `%` – Matches any sequence of characters.
    
- `_` – Matches a single character.

**Example (3-character name):**

```sql
`SELECT *  FROM employees WHERE name LIKE '___';`
```

## 5. `EXISTS`

- **What it does:** Filters based on whether a subquery returns any results.
    
- **Why it's useful:** More efficient than `IN` when checking for the existence of rows.
    
**Example:**

```sql
`SELECT *  FROM employees e WHERE EXISTS (     SELECT 1     FROM orders o     WHERE o.employee_id = e.employee_id );`
```

## 6. `DISTINCT` (vs `GROUP BY`)

- **What it does:** Removes duplicate rows from the result set.
    
- **Why it's useful:** Useful for getting unique values, but sometimes overlooked or misused. For aggregation, use `GROUP BY`.

**Example:**

```sql
`SELECT DISTINCT job_title  FROM employees;`
```

## 7. `HAVING` (vs `WHERE`)

- **What it does:** Filters aggregated results (used after `GROUP BY`).
    
- **Why it's useful:** Often mixed up with `WHERE`, but used for filtering aggregated data.

**Example:**
```sql
`SELECT department_id, COUNT(*) AS num_employees FROM employees GROUP BY department_id HAVING COUNT(*) > 2;`
```

## 8. `LIMIT` / `OFFSET`

- **What it does:** Limits the number of rows returned or skips a specified number of rows.
    
- **Why it's useful:** Useful for pagination or limiting the result set.

**Example (Limit to 10 rows):**

```sql
`SELECT *  FROM employees LIMIT 10;`
```

**Example (Limit with offset):**

```sql
`SELECT *  FROM employees LIMIT 10 OFFSET 10;`
```

