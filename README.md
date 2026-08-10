# Order Management Dashboard

## Project Overview

The Order Management Dashboard was created to help operational teams manage customer orders that are still waiting to be activated.

The solution provides a single operational view showing:

- Outstanding orders
- Which department owns the next action
- What action is required
- How long an order has remained with a team
- Operational blockers and exceptions

The most important part of the solution is the custom **Owner / Sub-Owner logic**.

Rather than simply displaying Salesforce statuses, I created business rules that translate multiple operational conditions into clear ownership and actions.

The solution has evolved alongside a major Salesforce transformation.

---

## Project Evolution

### OB2 — Salesforce Legacy

The first version, **OB2**, was developed around two years ago when customer orders were still mainly managed in Salesforce Legacy.

At that time, the newer Salesforce environment was primarily being used for installation activities.

OB2 combined information from several operational sources and used PostgreSQL as the main transformation layer before loading the final dataset into Power BI.

```text
Salesforce Legacy
        +
Installation / Operational Data
        ↓
PostgreSQL
        ↓
SQL Functions
        ↓
Business Logic
        ↓
Final Reporting Table
        ↓
Power BI
        ↓
OB2 Orderbook
