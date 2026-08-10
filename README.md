# Order Management Dashboard

## Project Overview

This project was created to help operational teams manage customer orders from order placement through to service activation.

The dashboard provides a clear view of:

- Outstanding orders
- Current stage of each order
- Which team owns the next action
- What action is required
- How long an order has remained within a stage
- Operational blockers and exceptions

The most important part of the solution is the custom **Owner / Sub-Owner logic**.

Instead of simply displaying Salesforce statuses, the solution combines information from multiple operational sources and converts it into clear business ownership and actions.

The project evolved alongside a major Salesforce transformation and now has two versions:

- **OB2** — built around Salesforce Legacy and PostgreSQL
- **OB3** — built around Salesforce Lightning and Power BI

---

## Business Problem

An order can pass through several teams before the customer's service becomes active.

The source systems contain a large amount of information, but the status of an order alone does not always answer the most important operational question:

> **Who needs to do what next?**

Teams may need to consider several conditions at the same time, including:

- Order status
- Installation progress
- Appointments
- Open support cases
- Work orders
- Property readiness
- Previous actions
- Operational exceptions

The challenge became even greater during the Salesforce transformation because active orders started existing across **two different platforms**.

The objective of the Orderbook was therefore not simply to report outstanding orders.

The objective was to transform complex operational data into **clear ownership and actionable information**.

---

# Project Evolution

## OB2 — Salesforce Legacy

The original Orderbook, **OB2**, was developed around two years ago when customer orders were still mainly managed in Salesforce Legacy.

At that time, the newer Salesforce environment was mainly being used for installation activities.

The data required for the report was available in PostgreSQL, so I designed a multi-stage SQL solution that combined several operational datasets before loading the final result into Power BI.

### OB2 Architecture

```text
Salesforce Legacy
        +
Installation / Operational Data
        ↓
PostgreSQL
        ↓
Multiple SQL Functions
        ↓
Data Transformation
        ↓
Ownership Business Logic
        ↓
Final Reporting Table
        ↓
Power BI
        ↓
OB2 Orderbook
```

The SQL layer handled most of the transformation and business logic, while Power BI was used for reporting, filtering and analysis.

---

## OB3 — Salesforce Lightning

In 2026, new customer orders started being created directly in **Salesforce Lightning**.

This meant customer orders were temporarily being managed across two systems:

```text
Existing Orders
Salesforce Legacy
        ↓
       OB2
```

```text
New Orders
Salesforce Lightning
        ↓
       OB3
```

I therefore needed to rebuild the Orderbook for the new platform.

The business requirement remained the same, but the technical architecture had to change.

Salesforce Lightning data was not available directly in the PostgreSQL database, so I created an automated extraction process using Power Automate and SharePoint.

### OB3 Architecture

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
Data Transformation
        ↓
Ownership Business Logic
        ↓
Power BI Data Model
        ↓
OB3 Orderbook
```

OB2 and OB3 currently operate as separate reports.

Once the remaining customers are migrated into Salesforce Lightning, OB2 will eventually become obsolete and OB3 will become the main Order Management solution.

---

# Business Ownership Logic

One of the key features of the Orderbook is the custom ownership model.

Every order is assigned two levels of classification:

### Master Owner

The high-level operational category responsible for progressing the order.

### Sub Owner

The specific scenario within that category.

The Sub Owner provides more detail about **why the order is currently there and what needs to happen next**.

---

## How the Logic Works

The ownership logic evaluates several pieces of information for each order.

```text
Order Status
        +
Installation Status
        +
Appointment Status
        +
Support Cases
        +
Work Orders
        +
Property Status
        +
Previous Actions
        ↓
Custom Business Logic
        ↓
Master Owner
        ↓
Sub Owner
        ↓
Required Action
```

For example, an order might be:

- Waiting for an installation appointment
- Blocked by a technical issue
- Waiting for customer information
- Waiting for property readiness
- Waiting for an internal data correction
- Already booked and requiring no immediate action

The Orderbook automatically identifies these scenarios and places the order into the appropriate operational category.

This allows teams to focus directly on the orders they can act on instead of manually reviewing multiple Salesforce records.

---

## Simplified Ownership Example

The real production logic contains many more conditions and has been anonymised for this portfolio.

| Master Owner | Example Sub Owner | Meaning | Typical Action |
|---|---|---|---|
| Installation | Booking Required | Order is ready but no appointment exists | Contact customer and book installation |
| Installation | Survey Required | A site survey is needed first | Arrange survey |
| Technical | Technical Issue | Technical work is blocking progress | Investigate and resolve issue |
| Operations | Data Fix Required | Incorrect or missing system data is blocking the order | Correct operational records |
| Customer | Customer Action | Customer confirmation or information is required | Contact customer |
| Scheduled | Installation Booked | Future appointment already exists | Monitor |
| Scheduled | Awaiting Activation | Service is scheduled to activate | No immediate action |
| Unassigned | New Scenario | Order does not match existing rules | Investigate and create new logic if required |

This classification is a simplified representation of the production model.

---

# My Approach

## 1. Understand the Operational Process

Before building the logic, I needed to understand how an order moves from sale through installation and activation.

I worked with stakeholders across different operational teams to understand:

- Their responsibilities
- Which orders required their attention
- Which statuses indicated a problem
- Which data fields they used
- What action should happen next

This allowed me to build the report around the real operational process rather than simply around the available source data.

---

## 2. Translate Business Knowledge into Data Logic

I converted operational rules into repeatable logic.

For example:

```text
IF
    order is ready for installation
AND
    no appointment exists

THEN
    Owner = Installation
    Sub Owner = Booking Required
```

Another simplified example:

```text
IF
    installation failed
AND
    technical issue exists

THEN
    Owner = Technical
    Sub Owner = Technical Investigation
```

This logic means users do not need to manually interpret multiple fields every time they review an order.

---

## 3. Build the Data Transformation

The transformation approach depends on the Salesforce platform.

### OB2

Most transformations are performed using:

- PostgreSQL
- SQL functions
- CTEs
- JOINs
- CASE statements
- Window functions

### OB3

Most transformations are performed using:

- Power Query
- Power BI
- DAX
- Power Automate
- SharePoint

The same business solution therefore demonstrates two different technical approaches.

---

## 4. Design for Operational Use

The final report was designed to answer simple operational questions:

- Which orders belong to my team?
- What needs attention?
- Why is the order blocked?
- What needs to happen next?
- How long has the order been waiting?
- Where are orders getting stuck?

---

# OB2 — SQL Development

OB2 was built using several PostgreSQL functions.

Instead of creating one very large query, the transformation was separated into logical stages.

```text
Customer Data
      ↓
Outstanding Orders
      ↓
Orderbook Step 1
      ↓
Orderbook Step 2
      ↓
Orderbook Step 3
      ↓
Orderbook Step 4
      ↓
Ownership Logic
      ↓
Final Reporting Table
      ↓
Power BI
```

Additional operational datasets were joined into the process when required.

```text
Support Cases ─────────────┐
                           │
Installation Work Orders ──┤
                           │
Survey Work Orders ────────┼──→ Orderbook Logic
                           │
Property Information ──────┤
                           │
Operational Exceptions ────┘
```

This structure made complex logic easier to manage, test and maintain.

---

# Simplified SQL Example

The following example is intentionally simplified and uses anonymised table and field names.

```sql
WITH latest_case AS (

    SELECT
        order_id,
        case_type,
        case_status,
        created_date,

        ROW_NUMBER() OVER (
            PARTITION BY order_id
            ORDER BY created_date DESC
        ) AS row_num

    FROM support_cases

    WHERE case_status <> 'Closed'
),

order_data AS (

    SELECT
        o.order_id,
        o.customer_id,
        o.order_status,
        o.created_date,
        i.install_status,
        i.appointment_date,
        c.case_type,

        COALESCE(
            i.install_status,
            'Unknown'
        ) AS install_status_clean

    FROM orders o

    LEFT JOIN installations i
        ON o.order_id = i.order_id

    LEFT JOIN latest_case c
        ON o.order_id = c.order_id
       AND c.row_num = 1

    WHERE o.activation_date IS NULL
),

classified_orders AS (

    SELECT
        *,

        CASE

            WHEN order_status = 'Pending'
                 AND appointment_date IS NULL
                 AND install_status_clean = 'Ready'
            THEN 'Installation'

            WHEN install_status_clean = 'Technical Issue'
            THEN 'Technical'

            WHEN order_status = 'On Hold'
                 AND case_type IS NOT NULL
            THEN 'Operations'

            WHEN appointment_date IS NOT NULL
            THEN 'Scheduled'

            ELSE 'Unassigned'

        END AS master_owner

    FROM order_data
)

SELECT
    order_id,
    customer_id,
    order_status,
    master_owner,
    created_date

FROM classified_orders;
```

---

## SQL Skills Demonstrated

- CTEs
- JOINs
- CASE statements
- Window functions
- `ROW_NUMBER()`
- `PARTITION BY`
- `COALESCE`
- Filtering
- Data cleansing
- Data transformation
- Multi-source integration
- Business-rule implementation

---

# Tracking Order Movement

I also created a **Sub-Owner Tracker**.

The tracker records how orders move between operational categories over time.

This allows the dashboard to analyse:

- Previous owner
- Current owner
- Date of ownership change
- Time spent within each stage
- Order movement
- Operational bottlenecks

A simplified SQL example:

```sql
SELECT
    order_id,
    snapshot_date,
    owner,

    LAG(owner) OVER (
        PARTITION BY order_id
        ORDER BY snapshot_date
    ) AS previous_owner,

    LAG(snapshot_date) OVER (
        PARTITION BY order_id
        ORDER BY snapshot_date
    ) AS previous_change_date

FROM order_history;
```

This information can then be used to measure how long orders remain within each operational stage.

---

# OB3 — Power BI Development

The move to Salesforce Lightning required a different technical solution.

Because the required Lightning data was not available directly in PostgreSQL, I created an automated reporting pipeline.

## Data Pipeline

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
Data Cleaning
        ↓
Dataset Merging
        ↓
Ownership Logic
        ↓
Power BI Model
        ↓
OB3 Dashboard
```

The automation removes the need for users to manually download Salesforce files and update the report source.

---

# Power Query

Power Query is used heavily in OB3 to prepare the reporting dataset.

The transformation includes:

- Connecting to multiple files
- Removing unnecessary columns
- Standardising field formats
- Handling NULL values
- Joining datasets
- Creating calculated fields
- Creating operational flags
- Applying ownership rules
- Preparing reporting tables

A simplified example:

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

# DAX

DAX is used for reporting measures and operational analysis.

### Outstanding Orders

```DAX
Outstanding Orders =
CALCULATE(
    COUNTROWS(Orders),
    Orders[Live Status] = FALSE()
)
```

### Average Order Age

```DAX
Average Order Age =
AVERAGEX(
    Orders,
    DATEDIFF(
        Orders[Order Date],
        TODAY(),
        DAY
    )
)
```

### Orders Requiring Action

```DAX
Orders Requiring Action =
CALCULATE(
    COUNTROWS(Orders),
    Orders[Action Required] = TRUE()
)
```

---

# Dashboard Design

The Orderbook contains a large amount of operational information.

Different departments need different fields to investigate their orders, so the dataset contains a large number of columns.

To keep the report usable, I introduced features including:

- Advanced filters
- Column selection
- Bookmarks
- Page navigation
- Owner / Sub-Owner views
- Operational KPIs
- Trend analysis
- Order-level investigation
- Geographic analysis

These features allow users to access detailed information without overcrowding the main report.

---

# Operational Analysis

The Orderbook is not only used to manage individual orders.

It also provides insight into the wider operational process.

## Ownership Analysis

The report can show:

- Number of orders assigned to each team
- Number of orders by Sub Owner
- Average time spent within each stage
- Changes in backlog
- Historical trends

---

## Order Movement

Using the Sub-Owner Tracker, the report can identify how orders move between different operational stages.

This helps identify:

- Bottlenecks
- Repeated issues
- Delays
- Areas requiring process improvement

---

## Geographic Analysis

Outstanding orders can also be analysed geographically.

This allows operational teams to identify areas with larger installation backlogs or specific operational issues.

---

# Supporting Automation

The Orderbook has also been used as the foundation for additional automation.

For example, I created workflows that identify orders that have recently moved into specific ownership categories.

```text
Orderbook Dataset
        ↓
Identify Ownership Changes
        ↓
Power Automate
        ↓
Generate Notification
        ↓
Operational Team
```

This allows teams to react to newly assigned work without continuously monitoring the dashboard.

---

# Tools Used

| Tool | Purpose |
|---|---|
| **Power BI** | Dashboard development and operational analysis |
| **Power Query** | OB3 data transformation |
| **DAX** | KPIs and reporting calculations |
| **PostgreSQL** | OB2 data processing |
| **SQL** | Data transformation and business logic |
| **Power Automate** | Salesforce extraction and operational automation |
| **SharePoint** | Staging Salesforce Lightning extracts |
| **Salesforce Legacy** | Original order management source |
| **Salesforce Lightning** | New order management platform |

---

# Business Impact

The Orderbook provides operational teams with a clear view of orders that still require action.

Instead of manually interpreting information across several Salesforce records and operational systems, users can quickly understand:

- Who owns the order
- Why they own it
- What stage the order has reached
- What is blocking progress
- What action needs to happen next
- How long the order has been waiting

This is particularly important during the Salesforce transformation because active customer orders currently exist across two different systems.

The Owner / Sub-Owner model creates a consistent operational framework across both platforms.

It helps departments manage workloads more effectively and makes complex operational data easier to act on.

---

# Skills Demonstrated

## SQL & Data Engineering

- PostgreSQL
- Complex SQL transformations
- CTEs
- JOINs
- CASE statements
- Window functions
- `ROW_NUMBER()`
- `LAG`
- `PARTITION BY`
- `COALESCE`
- Data cleansing
- Multi-stage data pipelines

## Power BI

- Power Query
- DAX
- Data modelling
- Dashboard design
- Interactive filtering
- Bookmarks
- Page navigation
- Operational KPIs

## Automation

- Power Automate
- Salesforce data extraction
- SharePoint integration
- Automated reporting pipelines
- Operational notifications

## Business Analysis

- Stakeholder engagement
- Process mapping
- Business-rule development
- Translating operational knowledge into technical logic
- Ownership modelling
- Operational problem solving
- Continuous improvement

---

# Project Journey

```text
2024
│
├── OB2 developed
│
├── Salesforce Legacy orders
│
├── PostgreSQL transformation
│
├── SQL ownership logic
│
└── Power BI reporting
│
│
2025
│
├── Ownership logic refined
│
├── Sub-Owner tracking added
│
├── Operational analysis expanded
│
└── Supporting automations developed
│
│
2026
│
├── New orders move to Salesforce Lightning
│
├── Orders now exist across two platforms
│
├── OB3 developed
│
├── Automated Salesforce extraction
│
├── SharePoint staging
│
├── Power Query transformation
│
└── Power BI ownership logic
│
▼
Future

Remaining customers migrate to Salesforce Lightning

        ↓

OB3 becomes the main Orderbook

        ↓

OB2 is retired
```

---

# Screenshots

## Order Management Dashboard

> Main operational view showing outstanding orders, ownership and key filters.

![Order Management Dashboard](images/orderbook-dashboard.png)

---

## Owner / Sub-Owner Analysis

> Analysis of orders by operational ownership and required action.

![Owner Analysis](images/owner-analysis.png)

---

## Operational Trends

> Historical view showing backlog and order movement across different operational teams.

![Operational Trends](images/order-trends.png)

---

## Order Investigation

> Detailed view used to investigate individual orders and operational blockers.

![Order Investigation](images/order-investigation.png)

---

## Solution Architecture

> Comparison between the SQL-driven OB2 architecture and the Power BI-driven OB3 architecture.

![Orderbook Architecture](images/orderbook-architecture.png)

---

# Key Takeaway

This project demonstrates how I combine **technical development with operational business knowledge**.

**OB2** demonstrates my SQL and PostgreSQL skills through a multi-stage data transformation pipeline.

**OB3** demonstrates my Power BI, Power Query and automation skills by rebuilding the same operational solution using a completely different data architecture.

Most importantly, the project demonstrates my ability to understand a complex business process and translate it into data logic that answers three simple operational questions:

> **Which orders require attention?**  
> **Who owns the next action?**  
> **What needs to happen next?**
