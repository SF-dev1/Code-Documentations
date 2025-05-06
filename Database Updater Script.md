# 🚀 Database Updater Script - README

## 📌 Overview
The **db_updater.php** script is designed to **read and execute SQL files** efficiently while handling **transactions, custom delimiters, and logging**. It supports **grouped queries** (`-- GROUP START` and `-- GROUP END`) to ensure **atomic execution**, meaning that if any query within a group fails, the transaction is rolled back.

## 🌟 Features
✅ **Transactional Execution**: Queries within `-- GROUP START` and `-- GROUP END` are executed as a single transaction.

✅ **Immediate Query Execution**: Queries outside a group are executed immediately.

✅ **Custom Delimiter Support**: Supports `DELIMITER $$ ... END $$` for stored procedures, triggers, and functions.

✅ **Error Handling & Logging**: Logs executed queries and captures failed queries for debugging.

✅ **Dynamic Environment Configuration**: Uses `.env` variables for database connection and file paths.

---

## 📂 File Format & Encoding Guidelines
- 📄 **File Extension**: The SQL file should have a `.sql` extension.
- 🔤 **Encoding**: Use `UTF-8` encoding to ensure compatibility with special characters.
- 🏗 **Line Endings**: Ensure consistent line endings (LF for Linux/macOS, CRLF for Windows).

---

## 📑 SQL File Structure
A properly structured SQL file should include:

### 📝 1. **Comments**
Use comments to explain different sections of the SQL file.

```sql
-- This is a single-line comment
/* This is a
multi-line comment */
```

### 📌 2. **Table Creation**

```sql
-- Creating users table
CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(255) NOT NULL
);
```

### 🔄 3. **Grouped Queries (Transactional Execution)**
All queries inside a `GROUP START` and `GROUP END` block run as a single transaction.
If any query fails, the entire block is rolled back.

```sql
-- GROUP START
INSERT INTO users (username) VALUES ('JohnDoe');
INSERT INTO users (username) VALUES ('JaneDoe');
-- GROUP END
```

### 🔄 4. **Create Triggers**
Create Trigger query inside a `TRIGGER START` and `TRIGGER END` block run as a single query.

```sql
-- TRIGGER START
CREATE TRIGGER trg_before_update_picklist_product
BEFORE UPDATE ON bas_inventory_picklist_products
FOR EACH ROW
BEGIN
    -- ✅ Update pending quantity
    SET NEW.pending_quantity = NEW.total_quantity - NEW.picked_quantity;

    -- ✅ Update status based on quantities
    IF NEW.picked_quantity = NEW.total_quantity THEN
        SET NEW.status = '2'; -- Completed
    ELSEIF NEW.picked_quantity > 0 AND NEW.picked_quantity <> NEW.total_quantity THEN
        SET NEW.status = '3'; -- Partially completed
    ELSE
        SET NEW.status = '0'; -- Pending
    END IF;
END;
-- TRIGGER END
```

### ⚡ 5. **Non-Grouped Queries (Immediate Execution)**
These queries execute immediately without rollback support.

```sql
INSERT INTO products (name) VALUES ('Laptop');
```

### 🔀 6. **Handling Custom Delimiters**
Use custom delimiters when defining stored procedures, triggers, or functions.
Restore the default delimiter (`;`) after execution.

```sql
DELIMITER $$
CREATE TRIGGER before_insert_users
BEFORE INSERT ON users
FOR EACH ROW
BEGIN
    SET NEW.username = UPPER(NEW.username);
END$$
DELIMITER ;
```

---

## ⚙️ Environment Configuration
The script relies on a `.env` file for database credentials and SQL file path.

### 📄 **Example `.env` File**
```
DB_HOST=localhost
DB_USER=root
DB_PASS=yourpassword
DB_NAME=mydatabase
DB_PORT=3306
SQL_FILE_PATH=/path/to/your/sqlfile.sql
```

---

## 🏃 How to Use the Script

### 📌 **1. Place Your SQL File**
Ensure your `.sql` file follows the guidelines above and is placed in the directory specified in `SQL_FILE_PATH`.

### ▶️ **2. Run the Script**
```sh
php db_updater.php
```

### 📜 **3. Check Logs**
If errors occur, check the log files at:
```
/var/www/skmienterprise/fims/log/{YEAR}/{MONTH}/{DAY}/
```
📌 Logs include:
- ✅ Successfully executed queries.
- ❌ Failed queries with error messages.

---

## ❓ Troubleshooting

### ⚠️ **Error: "This command is not supported in the prepared statement protocol yet"**
💡 **Solution**: The script now correctly handles `CREATE TRIGGER`, `CREATE PROCEDURE`, and `CREATE FUNCTION` using `multi_query()`.

### ⚠️ **Error: "GROUP END without GROUP START detected"**
💡 **Solution**: Ensure every `-- GROUP END` has a corresponding `-- GROUP START`.

### ⚠️ **Error: "SQL file not found"**
💡 **Solution**: Check `SQL_FILE_PATH` in `.env` and ensure the file exists.

---

## 🚀 Future Enhancements
🔹 Add a **CLI option** to specify a custom SQL file path.

🔹 Implement **unit tests** for testing different SQL scenarios.

🔹 Introduce **parallel execution** for improved performance.

---

## 🎯 Conclusion
This script provides a **robust solution** for executing SQL files in a structured, transactional, and error-logged manner. Following the provided **SQL formatting guidelines** ensures seamless execution and reliable database updates.

🛠 Happy coding! 🎉
