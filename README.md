# Order Management Dashboard

## Project Overview

The Orderbook is an operational Power BI solution used to track customer orders from sale through to service activation.

Its main purpose is simple:

> **Show which orders need attention, who owns the next action, and what needs to happen next.**

The solution has evolved as the business grew and moved from Salesforce Legacy to Salesforce Lightning.

---

## The Challenge

The order journey became increasingly complex as systems changed.

```text
Before 2024
Orders + Installations
        ↓
Salesforce Legacy
        ↓
Excel Orderbook
```

The original Orderbook was created in Excel when the customer base was much smaller.

In **2024**, the Salesforce transformation started.

Orders continued to be managed in Salesforce Legacy, but installations moved to the new Salesforce Field Service environment.

```text
2024

Salesforce Legacy
     Orders
       │
       ├──────────────┐
       │              │
       ↓              ↓
Customer Data       Salesforce FSL
                    Installations
       │              │
       └──────┬───────┘
              ↓
          PostgreSQL
              ↓
             OB2
              ↓
           Power BI
```

This created the need for a more scalable Orderbook capable of combining information from both systems.

---

## OB2 — SQL-Driven Orderbook

I rebuilt the Orderbook using **PostgreSQL and Power BI**.

Multiple SQL functions progressively combine:

- Customer orders
- Installation data
- Work orders
- Cases
- Property information
- Operational exceptions

The final dataset applies custom business logic before being loaded into Power BI.

```text
Multiple Data Sources
        ↓
PostgreSQL
        ↓
SQL Functions
        ↓
Business Logic
        ↓
Final Dataset
        ↓
Power BI
```

This replaced a spreadsheet-based process with a much more scalable reporting solution.

---

## 2026 — A New Challenge

In **2026**, Salesforce Lightning was introduced for order management.

Orders can now exist in either:

- Salesforce Legacy
- Salesforce Lightning

while installations continue to be managed in **Salesforce Field Service**.

```text
Salesforce Legacy ─────┐
                       │
Salesforce Lightning ──┼── Orders
                       │
Salesforce FSL ─────────┘
        Installations
```

This meant OB2 alone could no longer provide the full operational picture.

So I created **OB3**.

---

## OB3 — Power BI-Driven Orderbook

Salesforce Lightning data was not available directly in the PostgreSQL database.

I therefore created a different architecture.

```text
Salesforce Lightning
        ↓
Salesforce Reports
        ↓
Power Automate
        ↓
SharePoint
        ↓
Power Query
        ↓
Business Logic
        ↓
Power BI
        ↓
OB3
```

Instead of recreating the SQL pipeline, I rebuilt the transformation and ownership logic mainly inside **Power BI and Power Query**.

OB2 and OB3 currently run alongside each other.

Once all customers are migrated to Salesforce Lightning, OB2 will eventually be retired.

---

# Turning Data Into Action

The most important part of the Orderbook is not the dashboard itself.

It is the **business ownership logic** behind it.

Salesforce may tell us the status of an order, but operational teams need to know:

> **Who needs to act next?**

I created a custom **Master Owner / Sub Owner** model.

### Master Owner

Identifies the department responsible for progressing the order.

### Sub Owner

Identifies the specific scenario and expected next action.

```text
Order Status
Installation Status
Cases
Work Orders
Appointments
Property Status
        ↓
Business Rules
        ↓
Master Owner
        ↓
Sub Owner
        ↓
Next Action
```

This converts complex operational data into something teams can immediately act on.

---

## Simplified Example

| Owner | Scenario | Action |
|---|---|---|
| Installation | Booking Required | Contact customer and book installation |
| Technical | Technical Issue | Investigate and resolve blocker |
| Operations | Data Issue | Correct system records |
| Customer | Customer Action | Contact customer |
| Scheduled | Appointment Booked | Monitor |
| Unassigned | New Scenario | Investigate and update logic |

> The real production rules are more detailed and have been simplified for this portfolio.

---

# SQL Example

OB2 uses multiple SQL functions to progressively build the final dataset.

```sql
WITH latest_case AS (

    SELECT
        order_id,
        case_type,

        ROW_NUMBER() OVER (
            PARTITION BY order_id
            ORDER BY created_date DESC
        ) AS row_num

    FROM support_cases

    WHERE case_status <> 'Closed'
),

orders AS (

    SELECT
        o.order_id,
        o.customer_id,
        o.order_status,
        i.install_status,
        i.appointment_date,
        c.case_type

    FROM customer_orders o

    LEFT JOIN installations i
        ON o.order_id = i.order_id

    LEFT JOIN latest_case c
        ON o.order_id = c.order_id
       AND c.row_num = 1
)

SELECT
    order_id,
    customer_id,

    CASE
        WHEN install_status = 'Technical Issue'
            THEN 'Technical'

        WHEN appointment_date IS NOT NULL
            THEN 'Scheduled'

        WHEN order_status = 'Pending'
            THEN 'Operations'

        ELSE 'Unassigned'

    END AS owner

FROM orders;
```

### SQL Skills

`CTEs` · `JOINs` · `CASE` · `ROW_NUMBER()` · `PARTITION BY` · `COALESCE` · Window Functions · Data Transformation

---

# Tracking Order Movement

I also created an ownership tracker to understand how orders move through the process.

```sql
SELECT
    order_id,
    snapshot_date,
    owner,

    LAG(owner) OVER (
        PARTITION BY order_id
        ORDER BY snapshot_date
    ) AS previous_owner

FROM order_history;
```

This allows the report to identify:

- How long orders remain with each team
- Where orders are getting stuck
- Changes in operational backlog
- Process bottlenecks

---

# Power BI Development

OB3 demonstrates a different side of the solution.

Most of the transformation is now performed using:

- **Power Query** for data preparation
- **DAX** for measures and KPIs
- **Power BI** for modelling and reporting
- **Power Automate** for Salesforce extraction
- **SharePoint** as the staging layer

A simplified Power Query ownership rule:

```powerquery
= Table.AddColumn(
    PreviousStep,
    "Owner",
    each
        if [InstallStatus] = "Technical Issue" then "Technical"
        else if [AppointmentDate] <> null then "Scheduled"
        else if [OrderStatus] = "Pending" then "Operations"
        else "Unassigned"
)
```

---

# Business Impact

The Orderbook gives operational teams a clear and consistent way to manage outstanding customer orders.

Instead of manually checking multiple systems and interpreting dozens of fields, users can quickly see:

- **What needs attention**
- **Who owns it**
- **Why they own it**
- **What should happen next**
- **How long the order has been waiting**

This became especially important during the Salesforce transformation, where orders and installations were spread across multiple systems.

---

# Tools Used

| Tool | Purpose |
|---|---|
| **PostgreSQL / SQL** | OB2 transformation and business logic |
| **Power BI** | Reporting, modelling and operational analysis |
| **Power Query** | OB3 transformation |
| **DAX** | Measures and KPIs |
| **Power Automate** | Salesforce Lightning data extraction |
| **SharePoint** | OB3 staging layer |
| **Salesforce** | Orders, customers and installation data |

---

# Skills Demonstrated

**SQL & Data Engineering**  
PostgreSQL · CTEs · JOINs · CASE · Window Functions · Data Transformation

**Power BI**  
Power Query · DAX · Data Modelling · Dashboard Design · Operational Reporting

**Automation**  
Power Automate · Salesforce Extraction · SharePoint Integration

**Business Analysis**  
Process Mapping · Stakeholder Requirements · Business Rules · Ownership Logic · Operational Problem Solving

---

# Project Journey

```text
Before 2024
Excel Orderbook
Salesforce Legacy
        ↓

2024
Salesforce transformation begins
        ↓
Orders → Salesforce Legacy
Installations → Salesforce FSL
        ↓
OB2 created
PostgreSQL + Power BI
        ↓

2026
Salesforce Lightning introduced
        ↓
Orders → Legacy + Lightning
Installations → FSL
        ↓
OB3 created
Power Automate + SharePoint
Power Query + Power BI
        ↓

Future
Full migration to Salesforce Lightning
        ↓
OB3 becomes the main Orderbook
OB2 retired
```

---

# Screenshots

## Order Management Dashboard

![Order Management Dashboard](images/orderbook-dashboard.png)

## Owner / Sub Owner Analysis

![Owner Analysis](images/owner-analysis.png)

## Operational Trends

![Operational Trends](images/order-trends.png)

## Order Investigation

![Order Investigation](images/order-investigation.png)

## Architecture

![Orderbook Architecture](images/orderbook-architecture.png)

---

# Key Takeaway

This project shows how the same business problem can require very different technical solutions as systems evolve.

**OB2** showcases my SQL and PostgreSQL skills.

**OB3** showcases my Power BI, Power Query and automation skills.

Both are built around the same principle:

> **Transform complex operational data into clear ownership and action.**
