# 📦 Inventory Management System (IMS) - Process Overview

Welcome to the Inventory Management System (IMS) documentation. This document provides a comprehensive explanation of the entire process involved in managing inventory — from product arrival to final stocking in the warehouse.

---

## 🔁 Overall Process Flow

The inventory lifecycle in the IMS follows these major stages:

1. Product Inbound using GRN and LOT identification
2. Serial Number Generation for each product
3. Tagging process: Carton and Box labeling
4. Initial Quality Control (QC) check
5. Cooling period for verified products
6. Final QC check post-cooling
7. Storage and stocking in the warehouse (removal of carton dependency)

Each of these steps is described in detail below.

---

## 1. 🛬 Product Inbound (GRN + LOT)

When products are received at the facility, they must be entered into the system using two identifiers:

- **GRN (Goods Receipt Note)**: This is the receipt document that logs the arrival of goods into the warehouse. It contains all relevant delivery and supplier details.
- **LOT ID**: Each batch of goods is assigned a unique LOT ID, which helps in tracking and managing batches separately.

The system logs the details of the incoming products, including quantity, source, date, and specifications.

---

## 2. 🔢 Serial Number Generation

Once the products are successfully entered into the system, each item is assigned a unique serial number. The format for the serial number is as follows:

```
SF01SK000001
```

### Breakdown of Serial Number:
- **SF**: Static prefix used for all serial numbers to indicate it’s part of the standard inventory.
- **01**: Represents the LOT ID under which the product was received.
- **SK**: Represents the GRN ID associated with the product’s arrival.
- **000001**: A 6-digit unique serial number that auto-increments with each new product.

This serial number ensures traceability and uniqueness of every single product in the inventory.

---

## 3. 🏷️ Tagging Process (Carton and Box Labeling)

Once serial numbers are generated, the tagging process begins. This involves grouping products and assigning them to physical containers for better handling and traceability.

### 🔹 Carton Label Creation
- A **carton** label is generated for a specific number of products (referred to as quantity 'X').
- Each carton serves as a parent container that can hold multiple boxes.
- The carton label contains data like carton ID, associated LOT and GRN IDs, and the total quantity it holds.

### 🔸 Box Label Creation
- Inside each carton, **boxes** are labeled to further divide the products. Each box can contain a different quantity (referred to as 'Y').
- One **carton can contain multiple boxes**, and the combined total of all products in those boxes **must not exceed the total quantity specified in the carton**.
- Each box is also uniquely labeled and mapped in the system.

After successful labeling, the system updates the status of each product to `qc_pending`, which means the product is awaiting quality control inspection.

---

## 4. 🔍 Quality Control (QC) - Initial Check

In this stage, the QC team inspects every product individually to ensure there are no defects or issues.

### QC Outcome Scenarios:

- ✅ **If the product is found to be in perfect condition**:
  - The QC team updates the status to `qc_cooling`.
  - The product is then moved to a cooling area for the next phase.

- ❌ **If an issue is found in the product**:
  - The QC team marks the product as `qc_failed`.
  - The system automatically removes the product from its original **carton and box**.
  - The product is then transferred to a separate location called the **QC Failed Bin**, where it’s held without any box assignment.

This ensures defective products are segregated and do not continue through the regular process flow.

---

## 5. ❄️ Cooling Period (72 Hours)

Products that pass the initial QC inspection go through a **cooling phase**. This is a mandatory **72-hour period** where the products remain untouched to allow for any delayed detection of quality issues.

After 72 hours, the QC team performs a **second inspection**:

### Post-Cooling QC Outcomes:

- ✅ **If no issues are found**:
  - Product status is updated to `qc_verified`.
  - Product is ready to be moved to the stock room.

- ❌ **If issues are detected during the post-cooling check**:
  - Status is again updated to `qc_failed`.
  - Product is removed from its current grouping and moved to the **QC Failed Bin**, similar to the initial QC failure process.

---

## 6. 📍 Stock Room and Storage Logic (Previous Method)

In the earlier system design, products were stored using a **hierarchy of LOCATION > CARTON > BOX**:

- The warehouse (stock room) contains racks, each divided into partitions.
- Every partition has a **Location ID** for easy reference.
- Products were stored based on their carton and box, and the location had to be manually assigned and scanned.

This method created inefficiencies and complexities due to the dependency on **Carton IDs**, especially when products needed to be relocated or re-organized.

---

## 7. 🚀 New Inventory Stocking Process (Carton-Free System)

To streamline operations and reduce complexity, the new process **eliminates the dependency on cartons** in the stock room.

### ✅ New System Behavior:

- As soon as a product is marked `qc_verified`, the system:
  1. **Automatically assigns a Box and Location** where the product should be stored.
  2. Uses **FIFO (First-In-First-Out)** logic based on LOT IDs to ensure older inventory is stored first.

### 📦 When the Inventory Team Scans a Product:

- The system will display:
  - The **Box ID** into which the product should be placed.
  - The **Location ID** (partition in the rack) for that box.

- If the **expected location is full**:
  - The system evaluates **priority levels** of all products currently in that location.
  - It will:
    - Identify the **least-priority product** currently stored there.
    - Automatically update its location to a fallback **Godown Location**.
    - Assign the now-available stock room location to the high-priority, newly arriving product.

### 🔄 Manual Steps by Inventory Team:

1. Scan the verified product.
2. System displays expected Box ID and Location ID.
3. If fallback is triggered, team:
   - Moves the **least-priority product** to the fallback location.
   - Stores the new product in the preferred, now-available space.

### 🥇 Box Stacking Order:

- All boxes in the stock room must be **stacked based on the priority** of their products.
- This ensures that high-priority products are stored in optimal locations and are easily accessible.

---

## 📌 Process Summary Table

| Step | Description | System Status |
|------|-------------|----------------|
| 1 | Product Inbound using GRN and LOT | Recorded in system |
| 2 | Serial Number Generation | Unique ID per product |
| 3 | Tagging: Carton & Box Labeling | `qc_pending` |
| 4 | Initial QC Check | `qc_cooling` or `qc_failed` |
| 5 | Cooling Period (72 Hours) | Waiting phase |
| 6 | Post-Cooling QC | `qc_verified` or `qc_failed` |
| 7 | Auto Assignment of Box & Location | Based on FIFO |
| 8 | Storage in Stock Room (No Carton) | Based on Box Priority |

---

## 🛠️ Additional Notes

- Cartons are used **only** for temporary grouping during tagging and QC.
- Final inventory tracking is managed **exclusively** via Box ID and Location ID.
- The system automatically handles reassignments to minimize manual work and prevent errors.

---

## 📫 Contact

For further details, training, or support, please contact the IMS Administration Team.