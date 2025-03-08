# 📌 Reports Management System - MySQL Database

Welcome to the **Reports Management System**! This database is designed to handle various types of reports dynamically without relying on static schemas. 🚀

---

## 📖 Table of Contents
1. [📂 Database Schema Overview](#database-schema-overview)
2. [🛠️ CRUD Operations](#crud-operations)
   - [➕ Create a Report](#create-a-report)
   - [🔍 Fetch Report Data](#fetch-report-data)
   - [✏️ Update Report Data](#update-report-data)
   - [🗑️ Delete a Report](#delete-a-report)
   - [🔎 Find Reports by Type](#find-reports-by-type)
3. [📝 Sample Queries](#sample-queries)
4. [📌 Best Practices](#best-practices)
5. [📊 Example Report](#example-report)

---

## 📂 Database Schema Overview

The database consists of **four main tables**:

1️⃣ **`bas_reports`** - Stores metadata of reports.
```sql
CREATE TABLE bas_reports (
    report_id INT PRIMARY KEY AUTO_INCREMENT,
    report_type VARCHAR(128) NOT NULL COMMENT 'E.g., Shopify, QA, Finance, etc.',
    report_status CHAR(1) DEFAULT 'A' COMMENT 'A => Active, D => Disabled',
    report_created_date DATETIME DEFAULT CURRENT_TIMESTAMP,
    report_generation_time DATETIME NOT NULL,
    report_recurrence_type CHAR(2) DEFAULT 'D' COMMENT 'D => Daily, W => Weekly, M => Monthly, Y => Yearly, O => Once, WD => Weekdays'
);
```

2️⃣ **`bas_report_data`** - Defines the report's data structure.
```sql
CREATE TABLE bas_report_data (
    data_id INT PRIMARY KEY AUTO_INCREMENT,
    report_id INT NOT NULL,
    data_key VARCHAR(32) NOT NULL COMMENT 'E.g., orders, returns, assigned_count',
    data_query TEXT COMMENT 'Query to fetch data for report generation',
    data_status CHAR(1) DEFAULT 'A' COMMENT 'A => Active, D => Disabled',
    data_created_date DATETIME DEFAULT CURRENT_TIMESTAMP,
    data_updated_date DATETIME DEFAULT NULL ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (report_id) REFERENCES bas_reports(report_id) ON DELETE CASCADE
);
```

3️⃣ **`bas_report_entries`** - Stores key-value data for any report.
```sql
CREATE TABLE bas_report_entries (
    entry_id INT PRIMARY KEY AUTO_INCREMENT,
    report_id INT NOT NULL,
    data_id INT NOT NULL COMMENT 'Foreign key from bas_report_data',
    data_value VARCHAR(255) NOT NULL COMMENT 'Stores numeric/string values',
    FOREIGN KEY (report_id) REFERENCES bas_reports(report_id) ON DELETE CASCADE,
    FOREIGN KEY (data_id) REFERENCES bas_report_data(data_id) ON DELETE CASCADE
);
```

4️⃣ **`bas_report_list_entries`** - Stores list-based data (e.g., UIDs in QA Reports).
```sql
CREATE TABLE bas_report_list_entries (
    list_id INT PRIMARY KEY AUTO_INCREMENT,
    entry_id INT NOT NULL COMMENT 'Foreign key from bas_report_entries',
    data_value VARCHAR(64) NOT NULL COMMENT 'Stores one item from the list',
    FOREIGN KEY (entry_id) REFERENCES bas_report_entries(entry_id) ON DELETE CASCADE
);
```

---

## 📊 Example Report

Let's take an example of a **QA Report** generated on a given day:

### **Sample QA Report Data**
```json
{
    "qa_report": {
        "sangita": {
            "assigned": { "count": 50, "uids": ["SF45SK000001", "SF45SK000002"] },
            "repaired": { "count": 45, "uids": ["SF45SK000003", "SF45SK000004"] },
            "cr": { "count": 4, "uids": ["SF45SK000005"] },
            "liquidation": { "count": 1, "uids": ["SF45SK000006"] },
            "return": { "count": 5, "uids": ["SF45SK000007", "SF45SK000008"] }
        }
    }
}
```

### **How This Data is Stored in the Database?**

#### ➕ Insert Report Metadata
```sql
INSERT INTO bas_reports (report_type, report_generation_time, report_recurrence_type)
VALUES ('QA', NOW(), 'D');
SELECT LAST_INSERT_ID() INTO @report_id;
```

#### 📝 Insert Data Key Definition
```sql
INSERT INTO bas_report_data (report_id, data_key, data_query)
VALUES 
(@report_id, 'assigned_count', 'SELECT COUNT(*) FROM assigned_table WHERE report_id = @report_id');
SELECT LAST_INSERT_ID() INTO @data_id;
```

#### 📊 Insert Key-Value Data
```sql
INSERT INTO bas_report_entries (report_id, data_id, data_value)
VALUES (@report_id, @data_id, '50');
SELECT LAST_INSERT_ID() INTO @entry_id;
```

#### 📋 Insert List-Based Data (UIDs)
```sql
INSERT INTO bas_report_list_entries (entry_id, data_value)
VALUES 
(@entry_id, 'SF45SK000001'),
(@entry_id, 'SF45SK000002');
```

---

This refined structure ensures **hierarchical organization**, improves **data integrity**, and allows **dynamic report generation** based on SQL queries. 🚀

Would you like additional optimizations or sample report retrieval queries? 😊

