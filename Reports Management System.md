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

The database consists of **three main tables**:

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

2️⃣ **`bas_report_entries`** - Stores key-value data for any report.
```sql
CREATE TABLE bas_report_entries (
    entry_id INT PRIMARY KEY AUTO_INCREMENT,
    report_id INT NOT NULL,
    data_key VARCHAR(64) NOT NULL COMMENT 'E.g., orders, returns, assigned_count',
    data_value VARCHAR(255) NOT NULL COMMENT 'Stores numeric/string values',
    FOREIGN KEY (report_id) REFERENCES bas_reports(report_id) ON DELETE CASCADE
);
```

3️⃣ **`bas_report_list_entries`** - Stores list-based data (e.g., UIDs in QA Reports).
```sql
CREATE TABLE bas_report_list_entries (
    list_id INT PRIMARY KEY AUTO_INCREMENT,
    report_id INT NOT NULL,
    data_key VARCHAR(64) NOT NULL COMMENT 'E.g., assigned_uids, repaired_uids',
    data_value VARCHAR(64) NOT NULL COMMENT 'Stores one item from the list',
    FOREIGN KEY (report_id) REFERENCES bas_reports(report_id) ON DELETE CASCADE
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

#### 📝 Insert Key-Value Data
```sql
INSERT INTO bas_report_entries (report_id, data_key, data_value)
VALUES 
(@report_id, 'sangita_assigned_count', '50'),
(@report_id, 'sangita_repaired_count', '45'),
(@report_id, 'sangita_cr_count', '4'),
(@report_id, 'sangita_liquidation_count', '1'),
(@report_id, 'sangita_return_count', '5');
```

#### 📋 Insert List-Based Data (UIDs)
```sql
INSERT INTO bas_report_list_entries (report_id, data_key, data_value)
VALUES 
(@report_id, 'sangita_assigned_uids', 'SF45SK000001'),
(@report_id, 'sangita_assigned_uids', 'SF45SK000002'),
(@report_id, 'sangita_repaired_uids', 'SF45SK000003'),
(@report_id, 'sangita_repaired_uids', 'SF45SK000004'),
(@report_id, 'sangita_cr_uids', 'SF45SK000005'),
(@report_id, 'sangita_liquidation_uids', 'SF45SK000006'),
(@report_id, 'sangita_return_uids', 'SF45SK000007'),
(@report_id, 'sangita_return_uids', 'SF45SK000008');
```

---

This example illustrates how any report type can be **dynamically stored** in the database using structured tables. 🚀

Would you like additional queries or optimizations? 😊