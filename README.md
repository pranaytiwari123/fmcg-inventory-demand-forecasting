# 🍫 Mondelez — Finished Goods Inventory & Demand Forecasting Optimization

### 📦 FMCG Supply Chain | Demand Forecasting | Inventory Optimization | ABC Analysis | Reorder Automation

> An end-to-end **FMCG finished-goods inventory optimization model** designed around a Mondelez-style supply network, combining **SQL-based demand analytics, machine-learning forecasting, ABC classification, safety-stock optimization, reorder-point calculations, and automated inventory alerts**.

---

## 📌 Project Overview

Inventory management in FMCG is a balancing act:

**Too much inventory → higher holding cost, working capital, and expiry risk**

**Too little inventory → stockouts, lost sales, and poor customer service**

This project develops a data-driven inventory planning framework for **150 finished-goods SKUs** distributed across a simulated India FMCG network.

The model combines historical demand, SKU economics, lead time, demand variability, inventory position, and machine-learning forecasts to determine:

* Which SKUs require the highest inventory attention
* How much safety stock should be maintained
* When a SKU should be reordered
* How much should be ordered
* Which SKUs are most critical to service levels
* Which forecasting approach performs best
* How the optimized reorder policy can improve network fill rate

---

# 🎯 Business Objective

The primary objective is to improve the trade-off between:

**Service Level ↔ Inventory Availability ↔ Working Capital**

The project answers a practical supply-chain question:

> **"Given demand uncertainty, lead time, SKU importance, and current inventory, what should we stock, when should we reorder, and how much should we order?"**

---

# 🏭 Network Scope

| Parameter                     |                   Model Scope |
| ----------------------------- | ----------------------------: |
| Finished Goods SKUs           |                       **150** |
| Product Categories            |                         **5** |
| Plants                        |                         **5** |
| Regional Distribution Centers |                         **5** |
| Demand History                |  **90-day demand statistics** |
| Revenue Analysis              |                  **12-month** |
| Simulation Period             |                  **12 weeks** |
| Forecast Horizon              |                 **Next week** |
| Inventory Strategy            | **ABC-tiered reorder policy** |

### Product Categories

* 🍫 Chocolates
* 🍪 Biscuits & Wafers
* 🥤 Malted Beverages
* 🍬 Candy & Gum
* 🎁 Premium Gifting

---

# 🔄 End-to-End Project Workflow

```text
Historical Sales / Secondary Sales
             ↓
      SQL Demand Analysis
             ↓
   SKU-Level Demand Statistics
     (Average + Std. Dev.)
             ↓
        ABC Analysis
     Revenue-Based Ranking
             ↓
   Machine Learning Forecasting
             ↓
      Forecast Evaluation
             ↓
    Safety Stock Calculation
             ↓
      Reorder Point (ROP)
             ↓
     Current Inventory Check
             ↓
    Suggested Order Quantity
             ↓
       Reorder Alerts
             ↓
     Dashboard & KPI Insights
             ↓
    Fill-Rate Simulation
```

---

# 🧩 Project Architecture

The workbook is structured into interconnected analytical layers.

### 1️⃣ SKU Master

Contains the core SKU master data:

* SKU ID
* SKU Name
* Category
* Unit Cost
* Unit Price
* Lead Time
* MOQ
* Case Pack
* Shelf Life
* Plant
* Distribution Centers

This acts as the **master/reference table** for downstream calculations.

---

### 2️⃣ Demand Statistics

Demand statistics are calculated at SKU level using a trailing 90-day demand window.

Key metrics:

**Average Daily Demand**

```text
Average Daily Demand =
Total Demand / Number of Days Observed
```

**Demand Standard Deviation**

Measures the variability or uncertainty in daily demand.

Higher standard deviation → more uncertain demand → potentially higher safety-stock requirement.

The workbook identifies this layer as being sourced from the SQL demand-statistics view:

```text
v_sku_demand_stats_90d
```

based on secondary-sales / distributor-offtake data.

---

# 📊 3️⃣ ABC Analysis

ABC analysis prioritizes inventory based on **annual revenue contribution**.

### Methodology

1. Calculate 12-month units sold
2. Calculate 12-month revenue
3. Rank SKUs by revenue
4. Calculate cumulative revenue contribution
5. Assign ABC class

### Classification Logic

| Class | Cumulative Revenue | Inventory Priority |
| ----- | -----------------: | ------------------ |
| **A** |          Up to 80% | 🔴 Highest         |
| **B** |             80–95% | 🟠 Medium          |
| **C** |          Above 95% | 🟢 Lower           |

### Formula Logic

```excel
=RANK(Revenue, Revenue_Range)
```

Cumulative revenue:

```excel
=SUMPRODUCT((Rank_Range<=Current_Rank)*Revenue_Range)
 / SUM(Revenue_Range)
```

ABC classification:

```excel
=IF(Cumulative_Revenue<=0.8,"A",
   IF(Cumulative_Revenue<=0.95,"B","C"))
```

### Why ABC matters

Not every SKU should receive the same inventory policy.

An important/high-revenue SKU should generally receive tighter inventory control than a low-value SKU.

---

# 🤖 4️⃣ Demand Forecasting

Multiple forecasting approaches were benchmarked using **WMAPE (Weighted Mean Absolute Percentage Error)**.

### Models Compared

| Model                   |      WMAPE |
| ----------------------- | ---------: |
| Naive — Last Week       |     28.31% |
| Moving Average — 4 Week |     25.76% |
| Random Forest           | **20.94%** |
| Gradient Boosting       |     21.13% |

### 🏆 Winning Model: Random Forest

The Random Forest model achieved the lowest WMAPE among the tested approaches.

The model uses engineered demand features such as:

* Lag variables
* Rolling-window demand
* Promotion-calendar features
* Festive-season indicators
* Weekly SKU-level demand

### Forecasting principle

```text
Historical Demand
       +
Demand Patterns
       +
Promotions
       +
Seasonality / Festivals
       ↓
Machine Learning Model
       ↓
Next-Week Demand Forecast
```

The model was evaluated using a **time-based 8-week holdout**, avoiding random shuffling of time-series observations and reducing the risk of data leakage.

---

# 📦 5️⃣ Safety Stock Calculation

Safety stock protects against demand uncertainty during replenishment lead time.

The project uses:

```text
Safety Stock =
Z × σ × √Lead Time
```

Where:

* **Z** = service-level factor
* **σ** = standard deviation of daily demand
* **Lead Time** = replenishment lead time in days

### ABC-Based Service Levels

| ABC Class | Z Value |
| --------- | ------: |
| A         |    2.05 |
| B         |    1.65 |
| C         |    1.28 |

This creates a differentiated inventory strategy:

```text
A SKU → Higher service protection
B SKU → Moderate protection
C SKU → Lower protection
```

---

# 🎯 6️⃣ Reorder Point

The reorder point determines when replenishment should be triggered.

```text
Reorder Point =
(Expected Demand During Lead Time)
+
(Safety Stock)
```

Implemented as:

```text
ROP =
Average Daily Demand × Lead Time
+
Safety Stock
```

### Example

If:

```text
Average Daily Demand = 100 units
Lead Time = 5 days
Safety Stock = 200 units
```

Then:

```text
ROP = (100 × 5) + 200
    = 700 units
```

If inventory falls to **700 units or below**, the SKU becomes a reorder candidate.

---

# 🛒 7️⃣ Suggested Order Quantity

The model goes beyond simply saying **"REORDER"**.

It also calculates how much inventory should be ordered.

The model uses:

```text
Suggested Order Qty =
MAX(0,
    Reorder Point
    + 10 Days Cycle Stock
    - On-Hand Inventory)
```

The quantity is then converted into cases using the SKU's case-pack size.

```text
Suggested Order Qty (Cases)
=
ROUNDUP(
Suggested Order Qty (Units) / Case Pack,
0
)
```

This makes the output more operationally useful because planners can move from:

**"We need to reorder."**

to:

**"We need to reorder approximately X units / Y cases."**

---

# 🚨 8️⃣ Automated Reorder Alerts

The final inventory-control layer converts calculations into an actionable alert list.

Each alert contains:

* SKU
* Product Name
* Category
* ABC Class
* On-Hand Inventory
* Days of Cover
* Lead Time
* Next-Week Forecast
* Urgency
* Suggested Order Quantity
* Case Pack
* Suggested Order Quantity in Cases

### Example Alert Logic

```text
Current Inventory ≤ Reorder Point
              ↓
          REORDER NOW
              ↓
     Calculate Order Quantity
              ↓
        Rank by Urgency
```

The alert layer prioritizes SKUs with extremely low inventory coverage and high near-term demand.

---

# 📈 9️⃣ Fill-Rate Impact Simulation

The project compares a traditional/naive inventory-review policy with the optimized reorder policy.

### Network Simulation

**Before — Naive Weekly Review**

```text
Fill Rate = 74.59%
```

**After — ABC-Tiered + Forecast-Driven Reorder Policy**

```text
Fill Rate = 98.74%
```

### Improvement

```text
98.74% − 74.59%
= 24.15 percentage points
```

Approximately:

> **+24.2 percentage points improvement in simulated fill rate**

The simulation covers:

* 150 SKUs
* 5 DCs
* 12 weeks
* 30-day warm-up period
* Variable demand
* Promotional spikes
* Festive-season effects

---

# 📊 Dashboard

The executive dashboard brings the major outputs together.

### Key KPIs

* Total SKUs
* Network Fill Rate
* Forecast Error
* Reorder Alerts
* Critical Alerts
* Winning Forecast Model

### Current Model Outputs

| KPI                          |            Result |
| ---------------------------- | ----------------: |
| Total SKUs                   |           **150** |
| Fill Rate After Optimization |        **98.74%** |
| Fill Rate Before             |        **74.59%** |
| Improvement                  |     **+24.15 pp** |
| Forecast WMAPE               |        **20.94%** |
| Reorder Alerts               |           **123** |
| Critical Alerts              |            **43** |
| Best Model                   | **Random Forest** |

> These are model/simulation outputs for the project network and should not be interpreted as actual Mondelez operational data.

---

# 🛠️ Tools & Technologies

### Excel

Used for:

* Data organization
* SKU master
* ABC analysis
* Safety-stock calculations
* Reorder-point calculations
* Order-quantity calculations
* Alert generation
* Dashboarding
* KPI visualization

### SQL

Used conceptually/in the project workflow for:

* Demand extraction
* SKU-level aggregation
* 90-day demand statistics
* Secondary-sales / distributor-offtake analysis

Example analytical view:

```text
v_sku_demand_stats_90d
```

### Python

Used for the machine-learning forecasting layer.

Key concepts:

* Feature engineering
* Lag variables
* Rolling statistics
* Promotional features
* Festive-season features
* Time-series train/test split
* Random Forest
* Gradient Boosting
* WMAPE evaluation

### Machine Learning

```text
Random Forest
Naive Forecast
Moving Average
```

---

# 🗂️ Workbook Structure

| Sheet                | Purpose                               |
| -------------------- | ------------------------------------- |
| `README`             | Project documentation and methodology |
| `Dashboard`          | Executive KPI dashboard               |
| `SKU_Master`         | SKU master/reference data             |
| `Demand_Stats`       | 90-day demand statistics              |
| `ABC_Analysis`       | Revenue-based ABC classification      |
| `Reorder_Point_Calc` | Safety stock, ROP and order quantity  |
| `Fill_Rate_Impact`   | Before vs after simulation            |
| `Forecast_Model`     | Forecast-model comparison             |
| `Reorder_Alerts`     | Prioritized replenishment alerts      |

---

# 🔗 How the Sheets Connect

The model follows a dependency chain:

```text
SKU_Master
     │
     ├──────────────┐
     ↓              ↓
Demand_Stats     ABC_Analysis
     │              │
     └──────┬───────┘
            ↓
    Reorder_Point_Calc
            │
            ↓
     Reorder_Alerts
            │
            ↓
        Dashboard
```

The forecasting layer provides an additional demand signal that feeds the broader replenishment strategy and simulation.

###  MADE BY 
PRANAY NATH TIWARI
CHEMICAL ENGINEER
