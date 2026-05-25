# Power BI Procurement Analytics Report
## Purchase Order Activity 2022–2024 | Executive Dashboard Blueprint

---

## REPORT ARCHITECTURE OVERVIEW

| Page | Title | Core Question Answered | Feeds Into |
|------|-------|------------------------|------------|
| **Page 1** | Spend Command Center | "Where is our money going and is it under control?" | Identifies which categories/suppliers need scrutiny → leads to Page 2 |
| **Page 2** | Supplier & Contract Performance | "Are our suppliers and contracts delivering value?" | Flags delivery/quality failures and contract risks → leads to Page 3 |
| **Page 3** | Compliance, Savings & Risk | "Are we saving money and managing risk?" | Closes the loop: correlates spend patterns with compliance and savings outcomes |

**Narrative thread:** Page 1 surfaces WHERE money flows → Page 2 asks WHO is delivering it → Page 3 asks HOW well are we controlling it.

---

## DATA MODEL NOTES

Before building visuals, establish these relationships in Power Query / Model View:

| Table | Key Column | Relationship |
|-------|-----------|--------------|
| PO_Transactions | PO_Number (PK) | Fact table |
| Suppliers | Supplier_ID | Many-to-one from PO |
| Contracts | Contract_ID | Many-to-one from PO |
| Invoices | Invoice_ID, PO_Number | One-to-many from PO |
| Payments | Payment_ID, Invoice_ID | One-to-many from Invoice |
| Budget | Budget_ID, Category | Lookup from PO Category |
| Savings | Savings_ID, PO_Number | One-to-one or lookup |
| Date Table | Date (PK) | All date fields |

> **Required:** Create a dedicated Date dimension table (continuous dates 01-Jan-2022 to 31-Dec-2024) and relate all date fields (PO_Date, Delivery_Date, Invoice_Date, Payment_Date) as inactive relationships, activated in measures via USERELATIONSHIP().

---

## CALCULATED COLUMNS (Add to PO_Transactions table)

```dax
// 1. PO Age (days from PO date to today or delivery)
PO Age Days =
DATEDIFF(PO_Transactions[PO_Date], TODAY(), DAY)

// 2. Delivery Status
Delivery Status =
IF(
    PO_Transactions[Actual_Delivery_Date] <= PO_Transactions[Expected_Delivery_Date],
    "On Time",
    IF(
        ISBLANK(PO_Transactions[Actual_Delivery_Date]),
        "Pending",
        "Late"
    )
)

// 3. Days Late
Days Late =
IF(
    PO_Transactions[Actual_Delivery_Date] > PO_Transactions[Expected_Delivery_Date],
    DATEDIFF(PO_Transactions[Expected_Delivery_Date], PO_Transactions[Actual_Delivery_Date], DAY),
    0
)

// 4. Invoice-to-PO Variance
Invoice Variance =
PO_Transactions[Invoice_Amount] - PO_Transactions[PO_Amount]

// 5. Invoice Variance % 
Invoice Variance Pct =
DIVIDE(
    PO_Transactions[Invoice_Amount] - PO_Transactions[PO_Amount],
    PO_Transactions[PO_Amount],
    0
)

// 6. Payment Status
Payment Status =
IF(
    ISBLANK(PO_Transactions[Payment_Date]),
    IF(PO_Transactions[Due_Date] < TODAY(), "Overdue", "Pending"),
    "Paid"
)

// 7. Compliant PO Flag (PO backed by valid contract)
Is Compliant =
IF(
    NOT(ISBLANK(PO_Transactions[Contract_ID])) &&
    PO_Transactions[PO_Date] >= RELATED(Contracts[Contract_Start_Date]) &&
    PO_Transactions[PO_Date] <= RELATED(Contracts[Contract_End_Date]),
    1, 0
)

// 8. Spend Tier (for segmentation)
Spend Tier =
SWITCH(
    TRUE(),
    PO_Transactions[PO_Amount] >= 100000, "Strategic (≥$100K)",
    PO_Transactions[PO_Amount] >= 25000,  "Tactical ($25K–$99K)",
    PO_Transactions[PO_Amount] >= 5000,   "Operational ($5K–$24K)",
    "Tail Spend (<$5K)"
)

// 9. Contract Expiry Risk (days to contract expiry from PO date)
Days to Contract Expiry =
DATEDIFF(
    PO_Transactions[PO_Date],
    RELATED(Contracts[Contract_End_Date]),
    DAY
)

// 10. Savings Realized Flag
Savings Realized =
IF(PO_Transactions[Actual_Cost] < PO_Transactions[Baseline_Cost], 1, 0)
```

---

## MEASURES LIBRARY (Core DAX)

```dax
// ─────────────────────────────────────────────
// SPEND & VOLUME
// ─────────────────────────────────────────────

Total Spend =
SUM(PO_Transactions[PO_Amount])

Total PO Count =
COUNTROWS(PO_Transactions)

Avg PO Value =
DIVIDE([Total Spend], [Total PO Count])

YTD Spend =
TOTALYTD([Total Spend], 'Date'[Date])

PY Spend =
CALCULATE([Total Spend], SAMEPERIODLASTYEAR('Date'[Date]))

YoY Spend Growth % =
DIVIDE([Total Spend] - [PY Spend], [PY Spend], 0)

Spend vs Budget =
[Total Spend] - SUM(Budget[Budget_Amount])

Budget Utilization % =
DIVIDE([Total Spend], SUM(Budget[Budget_Amount]), 0)

// ─────────────────────────────────────────────
// SUPPLIER PERFORMANCE
// ─────────────────────────────────────────────

On Time Delivery Rate =
DIVIDE(
    COUNTROWS(FILTER(PO_Transactions, PO_Transactions[Delivery Status] = "On Time")),
    COUNTROWS(FILTER(PO_Transactions, PO_Transactions[Delivery Status] <> "Pending")),
    0
)

Avg Days Late =
AVERAGEX(
    FILTER(PO_Transactions, PO_Transactions[Days Late] > 0),
    PO_Transactions[Days Late]
)

Supplier Count =
DISTINCTCOUNT(PO_Transactions[Supplier_ID])

Active Suppliers =
CALCULATE(
    DISTINCTCOUNT(PO_Transactions[Supplier_ID]),
    PO_Transactions[PO_Date] >= DATE(YEAR(TODAY()), 1, 1)
)

Top 10 Supplier Spend =
CALCULATE(
    [Total Spend],
    TOPN(10, ALL(Suppliers[Supplier_Name]), [Total Spend], DESC)
)

Supplier Spend Concentration % =
DIVIDE([Top 10 Supplier Spend], [Total Spend], 0)

// ─────────────────────────────────────────────
// CONTRACT MANAGEMENT
// ─────────────────────────────────────────────

Contract Coverage % =
DIVIDE(
    CALCULATE([Total Spend], PO_Transactions[Is Compliant] = 1),
    [Total Spend],
    0
)

Off-Contract Spend =
CALCULATE([Total Spend], PO_Transactions[Is Compliant] = 0)

Contracts Expiring in 90 Days =
CALCULATE(
    DISTINCTCOUNT(Contracts[Contract_ID]),
    Contracts[Contract_End_Date] >= TODAY(),
    Contracts[Contract_End_Date] <= TODAY() + 90
)

// ─────────────────────────────────────────────
// INVOICE & PAYMENT
// ─────────────────────────────────────────────

Total Invoiced =
SUM(PO_Transactions[Invoice_Amount])

Invoice Accuracy Rate =
DIVIDE(
    COUNTROWS(FILTER(PO_Transactions, ABS(PO_Transactions[Invoice Variance Pct]) <= 0.05)),
    COUNTROWS(FILTER(PO_Transactions, NOT(ISBLANK(PO_Transactions[Invoice_Amount])))),
    0
)

Total Invoice Variance =
SUMX(PO_Transactions, PO_Transactions[Invoice Variance])

Overdue Payments =
CALCULATE([Total Spend], PO_Transactions[Payment Status] = "Overdue")

Avg Payment Days =
AVERAGEX(
    FILTER(PO_Transactions, NOT(ISBLANK(PO_Transactions[Payment_Date]))),
    DATEDIFF(PO_Transactions[Invoice_Date], PO_Transactions[Payment_Date], DAY)
)

// ─────────────────────────────────────────────
// SAVINGS
// ─────────────────────────────────────────────

Total Savings =
SUMX(
    FILTER(PO_Transactions, PO_Transactions[Savings Realized] = 1),
    PO_Transactions[Baseline_Cost] - PO_Transactions[Actual_Cost]
)

Savings Rate % =
DIVIDE([Total Savings], SUM(PO_Transactions[Baseline_Cost]), 0)

Savings vs Target =
[Total Savings] - SUM(Savings[Savings_Target])

// ─────────────────────────────────────────────
// COMPLIANCE & RISK
// ─────────────────────────────────────────────

Compliance Rate =
DIVIDE(
    CALCULATE([Total PO Count], PO_Transactions[Is Compliant] = 1),
    [Total PO Count],
    0
)

High Risk PO Count =
CALCULATE(
    [Total PO Count],
    PO_Transactions[Is Compliant] = 0,
    PO_Transactions[PO_Amount] >= 25000
)

Tail Spend % =
DIVIDE(
    CALCULATE([Total Spend], PO_Transactions[Spend Tier] = "Tail Spend (<$5K)"),
    [Total Spend],
    0
)
```

---

---

# PAGE 1: SPEND COMMAND CENTER

**Executive Question:** *"Where is our money going, is it growing or shrinking, and are we within budget?"*

**Transition to Page 2:** KPI cards revealing supplier concentration or off-contract spend invite the user to drill into supplier and contract performance on Page 2.

---

## PAGE 1 LAYOUT

```
┌─────────────────────────────────────────────────────────────────────┐
│  HEADER: Filters — Year | Quarter | Category | Supplier             │
├────────┬────────┬────────┬────────┬────────────────────────────────┤
│ KPI 1  │ KPI 2  │ KPI 3  │ KPI 4  │  KPI 5                         │
│ Total  │ Total  │ Budget │ YoY    │  Off-Contract                  │
│ Spend  │ PO Cnt │ Util % │ Growth │  Spend $                       │
├────────┴────────┴────────┴────────┴────────────────────────────────┤
│                                                                     │
│  [VISUAL 1 — 60%]          │  [VISUAL 2 — 40%]                     │
│  Spend Trend by Month      │  Spend by Category (Treemap)          │
│  (Line + Area)             │                                       │
│                            │                                       │
├────────────────────────────┴───────────────────────────────────────┤
│                                                                     │
│  [VISUAL 3 — 50%]          │  [VISUAL 4 — 50%]                     │
│  Top 10 Suppliers by Spend │  Spend vs Budget by Category          │
│  (Bar Chart)               │  (Clustered Bar)                      │
│                            │                                       │
└─────────────────────────────────────────────────────────────────────┘
```

---

## PAGE 1 — VISUALS & MEASURES

---

### KPI Cards (Row of 5 cards — top of page)

**Purpose:** Give executives the full picture in 5 numbers before they look at any chart.

| Card | Measure | Secondary Metric | Conditional Formatting |
|------|---------|-----------------|------------------------|
| Total Spend | `Total Spend` | `YoY Spend Growth %` | Arrow ↑↓ with % change |
| PO Count | `Total PO Count` | `Avg PO Value` | — |
| Budget Utilization | `Budget Utilization %` | `Spend vs Budget` ($) | Red if >100%, Amber 90-100%, Green <90% |
| YoY Growth | `YoY Spend Growth %` | `PY Spend` for context | Red if >10% unexplained growth |
| Off-Contract Spend | `Off-Contract Spend` | % of Total Spend | Red if >15% of total |

---

### Visual 1 — Monthly Spend Trend (Line + Area Chart)

**Business Question:** *Is total spending accelerating, decelerating, or seasonal? Are there anomalous spikes?*

**Chart type:** Area chart (shaded) with a line for Budget reference

**X-axis:** Month-Year (Jan 2022 – Dec 2024)
**Y-axis:** Spend ($)

**Series:**
- `Total Spend` (primary — area fill, blue)
- Monthly Budget line (flat reference — dashed, gray)
- `YTD Spend` running total (secondary axis, optional)

**Measures used:**
```dax
// Monthly Budget Reference Line
Monthly Budget =
DIVIDE(SUM(Budget[Budget_Amount]), 12)

// Spend Rolling 3-Month Average (smoothing)
Spend 3M Rolling Avg =
AVERAGEX(
    DATESINPERIOD('Date'[Date], LASTDATE('Date'[Date]), -3, MONTH),
    [Total Spend]
)
```

**Annotations:** Use reference lines for COVID-period supply shocks (2022) and any contract renewal periods.

**Interaction:** Click any month → filters all Page 1 visuals AND passes selection to Page 2/3 via cross-page filtering (set "Edit interactions" accordingly).

---

### Visual 2 — Spend by Category (Treemap)

**Business Question:** *Which spend categories dominate, and which are growing disproportionately?*

**Chart type:** Treemap

**Values:** `Total Spend`
**Category:** `Category` (primary) > `Sub-Category` (drill-down)
**Tooltip:** `Total Spend`, `YoY Spend Growth %`, `Budget Utilization %`, `PO Count`

**Conditional coloring:**
- Categories over budget → Red family
- Categories within 10% of budget → Amber
- Categories well within budget → Teal/Green

**Measures used:**
```dax
// Category Budget Variance for color
Category Budget Variance % =
DIVIDE(
    [Total Spend] - SUM(Budget[Budget_Amount]),
    SUM(Budget[Budget_Amount]),
    0
)
```

**Why treemap over pie:** Proportional area encoding handles 8–15 categories cleanly; pie charts become unreadable above 5 segments.

---

### Visual 3 — Top 10 Suppliers by Spend (Horizontal Bar)

**Business Question:** *Who are we most financially dependent on? Is supplier concentration a risk?*

**Chart type:** Horizontal bar chart (ranked)

**Y-axis:** Supplier Name
**X-axis:** `Total Spend`
**Secondary bar:** `On Time Delivery Rate` (overlaid as dots or a small secondary bar — dual encoding)
**Color:** Gradient from highest to lowest spend

**Measures used:**
- `Total Spend`
- `On Time Delivery Rate`
- `Supplier Spend Concentration %` (shown in subtitle)
- `PO Count` (shown in tooltip)

```dax
// Cumulative spend % for Pareto annotation
Cumulative Spend % =
VAR CurrentSupplier = MAX(Suppliers[Supplier_Name])
VAR CurrentSpend = [Total Spend]
VAR TotalSpend = CALCULATE([Total Spend], ALL(Suppliers))
RETURN
DIVIDE(
    SUMX(
        FILTER(
            ALL(Suppliers[Supplier_Name]),
            CALCULATE([Total Spend]) >= CurrentSpend
        ),
        CALCULATE([Total Spend])
    ),
    TotalSpend
)
```

**Design note:** Add a vertical reference line at 80% cumulative spend to visually identify Pareto suppliers (the 20% who account for 80% of spend).

---

### Visual 4 — Spend vs Budget by Category (Clustered Bar)

**Business Question:** *Which categories are over budget and by how much?*

**Chart type:** Clustered Bar (Actual vs Budget side by side) OR a single bar with a budget line overlay

**Y-axis:** Category
**X-axis:** Amount ($)
**Series 1:** `Total Spend` (blue)
**Series 2:** Budget target (orange reference line or bar)

**Conditional formatting on data labels:** Red label if spend > budget

**Measures used:**
- `Total Spend`
- `SUM(Budget[Budget_Amount])`
- `Spend vs Budget` (shown in tooltip as $ overage/underage)
- `Budget Utilization %`

---

---

# PAGE 2: SUPPLIER & CONTRACT PERFORMANCE

**Executive Question:** *"Are our suppliers delivering on time? Are our contracts providing value and coverage? Where are the relationship risks?"*

**Transition from Page 1:** If you clicked a supplier on Page 1's bar chart, Page 2 will be filtered to that supplier, showing their specific delivery track record and contract health.

**Transition to Page 3:** Contract compliance gaps identified here link directly to the compliance risk view on Page 3.

---

## PAGE 2 LAYOUT

```
┌─────────────────────────────────────────────────────────────────────┐
│  HEADER: Filters — Year | Supplier | Category | Contract Status     │
├────────┬────────┬────────┬────────┬────────────────────────────────┤
│ KPI 1  │ KPI 2  │ KPI 3  │ KPI 4  │ KPI 5                          │
│ On-Time│ Avg    │Contract│ Expiring│ Invoice                        │
│ Rate % │ Days   │ Cov %  │ 90 days │ Accuracy %                     │
│        │ Late   │        │         │                               │
├────────┴────────┴────────┴────────┴────────────────────────────────┤
│                                                                     │
│  [VISUAL 1 — 55%]          │  [VISUAL 2 — 45%]                     │
│  Supplier Scorecard        │  Delivery Performance Trend           │
│  (Matrix / Table)          │  (Line Chart by Month)                │
│                            │                                       │
├────────────────────────────┴───────────────────────────────────────┤
│                                                                     │
│  [VISUAL 3 — 50%]          │  [VISUAL 4 — 50%]                     │
│  Contract Coverage &       │  Invoice Accuracy &                   │
│  Expiry Timeline           │  Payment Cycle                        │
│  (Gantt-style bar)         │  (Scatter or Clustered Bar)           │
│                            │                                       │
└─────────────────────────────────────────────────────────────────────┘
```

---

## PAGE 2 — VISUALS & MEASURES

---

### KPI Cards (Row of 5 — top of page)

| Card | Measure | Secondary | Alert Threshold |
|------|---------|-----------|-----------------|
| On-Time Delivery Rate | `On Time Delivery Rate` | vs prior year | Red if <85% |
| Avg Days Late | `Avg Days Late` | # of late POs | Red if >7 days |
| Contract Coverage | `Contract Coverage %` | Off-contract $ | Red if <80% |
| Contracts Expiring 90d | `Contracts Expiring in 90 Days` | Count + Value at risk | Red if count > 0 |
| Invoice Accuracy | `Invoice Accuracy Rate` | Total invoice variance $ | Red if <90% |

---

### Visual 1 — Supplier Scorecard (Matrix Visual)

**Business Question:** *"Which specific suppliers are underperforming across the dimensions that matter — delivery, spend, invoice accuracy?"*

**Chart type:** Matrix (table with conditional formatting)

**Rows:** Supplier Name
**Columns (measures):**

| Column | Measure | Conditional Format |
|--------|---------|-------------------|
| Total Spend | `Total Spend` | Data bar |
| PO Count | `Total PO Count` | — |
| On-Time Rate | `On Time Delivery Rate` | Red <80%, Amber 80–90%, Green >90% |
| Avg Days Late | `Avg Days Late` | Red >10d, Amber 5–10d |
| Invoice Accuracy | `Invoice Accuracy Rate` | Red <85% |
| Contract Coverage | Supplier-level coverage % | Red <70% |
| YoY Spend Δ | `YoY Spend Growth %` | Arrow indicator |

```dax
// Supplier-level Invoice Accuracy
Supplier Invoice Accuracy =
DIVIDE(
    CALCULATE(
        COUNTROWS(PO_Transactions),
        ABS(PO_Transactions[Invoice Variance Pct]) <= 0.05
    ),
    CALCULATE(
        COUNTROWS(PO_Transactions),
        NOT(ISBLANK(PO_Transactions[Invoice_Amount]))
    ),
    0
)

// Supplier Risk Score (composite: weight delivery + compliance + invoice)
Supplier Risk Score =
VAR DeliveryScore = (1 - [On Time Delivery Rate]) * 40
VAR ComplianceScore = (1 - [Contract Coverage %]) * 35
VAR InvoiceScore = (1 - [Invoice Accuracy Rate]) * 25
RETURN ROUND(DeliveryScore + ComplianceScore + InvoiceScore, 1)
```

**Design note:** Sort by `Supplier Risk Score` descending by default so highest-risk suppliers appear first. Add a slicer to filter to "Critical Suppliers" (top 10 by spend).

---

### Visual 2 — Delivery Performance Trend (Line Chart)

**Business Question:** *"Is our overall on-time delivery improving or deteriorating over time? Are problems seasonal or structural?"*

**Chart type:** Line chart with two series

**X-axis:** Month-Year
**Y-axis (left):** `On Time Delivery Rate` (%)
**Y-axis (right):** `Avg Days Late` (days)

**Reference line:** 90% on-time target line (horizontal, dashed)

**Drill-through:** Click any month to drill to individual POs that were late.

**Measures used:**
```dax
// Monthly On-Time Rate (for trend)
Monthly OTD Rate =
CALCULATE(
    [On Time Delivery Rate],
    DATESMTD('Date'[Date])
)

// Late PO Volume (for secondary context)
Late PO Count =
CALCULATE(
    [Total PO Count],
    PO_Transactions[Delivery Status] = "Late"
)
```

---

### Visual 3 — Contract Coverage & Expiry (Gantt-style Timeline)

**Business Question:** *"Which contracts are expiring soon? Which spend is at risk of falling off-contract?"*

**Chart type:** Gantt bar chart (built using a stacked bar visual or a custom Gantt visual from AppSource)

**Y-axis:** Contract Name / Supplier
**X-axis:** Date (2022–2025)
**Bar length:** Contract duration (Start Date to End Date)
**Color encoding:**
- Green: Active, >90 days remaining
- Amber: Expiring within 90 days
- Red: Expired (spend still occurring against it)
- Gray: Expired, no active spend

**Spend at risk annotation:** Contracts expiring in <90 days should show total annual spend value in the data label.

**Measures used:**
```dax
// Spend at Risk (from expiring contracts)
Spend at Risk =
CALCULATE(
    [Total Spend],
    Contracts[Contract_End_Date] <= TODAY() + 90,
    Contracts[Contract_End_Date] >= TODAY()
)

// Off-Contract Spend Rate (monthly)
Off-Contract Rate =
DIVIDE(
    CALCULATE([Total Spend], PO_Transactions[Is Compliant] = 0),
    [Total Spend],
    0
)
```

---

### Visual 4 — Invoice & Payment Performance (Clustered Bar)

**Business Question:** *"Are we being invoiced accurately and paying on time? Where are we losing money to invoice discrepancies?"*

**Chart type:** Clustered Bar (Invoice Variance by Category) + small KPI tiles for payment cycle

**Y-axis:** Category or Supplier
**X-axis:** Invoice Variance ($) — show both positive (overbilled) and negative (underbilled) bars
**Color:** Red for overbilling (supplier billed more than PO), Blue for underbilling

**Measures used:**
```dax
// Total Overbilling
Total Overbilling =
SUMX(
    FILTER(PO_Transactions, PO_Transactions[Invoice Variance] > 0),
    PO_Transactions[Invoice Variance]
)

// Avg Days to Pay (payment cycle efficiency)
Avg Days to Pay =
AVERAGEX(
    FILTER(PO_Transactions, NOT(ISBLANK(PO_Transactions[Payment_Date]))),
    DATEDIFF(PO_Transactions[Invoice_Date], PO_Transactions[Payment_Date], DAY)
)

// Payment within terms (e.g., Net 30)
Payment Within Terms % =
DIVIDE(
    CALCULATE(
        [Total PO Count],
        PO_Transactions[Payment Status] = "Paid",
        DATEDIFF(PO_Transactions[Invoice_Date], PO_Transactions[Payment_Date], DAY) <= 30
    ),
    CALCULATE([Total PO Count], PO_Transactions[Payment Status] = "Paid"),
    0
)
```

---

---

# PAGE 3: COMPLIANCE, SAVINGS & RISK

**Executive Question:** *"Are we following procurement rules, realizing our cost savings, and where are the biggest operational risks?"*

**Transition from Page 2:** Page 2 showed which suppliers have low contract coverage. Page 3 quantifies the COST of that non-compliance and maps it alongside savings performance — answering whether efficiency gains are being offset by compliance failures.

---

## PAGE 3 LAYOUT

```
┌─────────────────────────────────────────────────────────────────────┐
│  HEADER: Filters — Year | Category | Compliance Status              │
├────────┬────────┬────────┬────────┬────────────────────────────────┤
│ KPI 1  │ KPI 2  │ KPI 3  │ KPI 4  │  KPI 5                         │
│Complian│ Total  │ Savings│Savings │  High-Risk                     │
│ce Rate │ Savings│ Rate % │vs Target│  PO Count                     │
├────────┴────────┴────────┴────────┴────────────────────────────────┤
│                                                                     │
│  [VISUAL 1 — 55%]          │  [VISUAL 2 — 45%]                     │
│  Compliance Rate Trend     │  Savings Waterfall                    │
│  + Off-Contract Spend      │  (Waterfall Chart)                    │
│  (Dual-axis Line)          │                                       │
│                            │                                       │
├────────────────────────────┴───────────────────────────────────────┤
│                                                                     │
│  [VISUAL 3 — 50%]          │  [VISUAL 4 — 50%]                     │
│  Risk Matrix               │  Tail Spend Analysis                  │
│  (Scatter: Spend vs Risk)  │  (Donut + Bar)                        │
│                            │                                       │
└─────────────────────────────────────────────────────────────────────┘
```

---

## PAGE 3 — VISUALS & MEASURES

---

### KPI Cards (Row of 5 — top of page)

| Card | Measure | Secondary | Alert |
|------|---------|-----------|-------|
| Compliance Rate | `Compliance Rate` | # Non-compliant POs | Red if <85% |
| Total Savings | `Total Savings` | vs same period last year | — |
| Savings Rate | `Savings Rate %` | Industry benchmark: 5–8% | Amber if <5% |
| Savings vs Target | `Savings vs Target` | % of target achieved | Red if negative |
| High-Risk POs | `High Risk PO Count` | Total value of high-risk POs | Red if >0 |

---

### Visual 1 — Compliance Trend + Off-Contract Spend (Dual-Axis Line)

**Business Question:** *"Is compliance improving over time? Is off-contract spend growing as a systemic problem?"*

**Chart type:** Dual-axis line chart

**X-axis:** Month-Year (2022–2024)
**Y-axis Left:** `Compliance Rate` (%) — line in green
**Y-axis Right:** `Off-Contract Spend` ($) — line in red
**Reference line:** 90% compliance target (horizontal dashed)

**Insight this enables:** If compliance rate drops and off-contract spend rises simultaneously, it signals a systemic process breakdown — not random. If they diverge (compliance holds but off-contract spend rises), it may indicate contract value vs. threshold misalignment.

**Measures used:**
```dax
// Rolling 3-month compliance rate
Compliance 3M Rolling =
AVERAGEX(
    DATESINPERIOD('Date'[Date], LASTDATE('Date'[Date]), -3, MONTH),
    [Compliance Rate]
)

// Non-compliant PO count trend
Non-Compliant PO Count =
CALCULATE(
    [Total PO Count],
    PO_Transactions[Is Compliant] = 0
)
```

---

### Visual 2 — Savings Waterfall (Waterfall Chart)

**Business Question:** *"What drove our savings — and what offset them? Are we achieving net positive procurement savings?"*

**Chart type:** Waterfall Chart

**Structure:**
- Starting baseline: `SUM(PO_Transactions[Baseline_Cost])` (total if no savings were achieved)
- Positive bars (savings drivers): Negotiated price reduction, Volume discounts, Early payment discounts, Contract consolidation savings
- Negative bars (savings offsets): Invoice overbilling, Maverick/off-contract premium spend, Price escalation
- Net result: Actual Total Spend

**Measures used:**
```dax
// Savings by type (requires Savings[Savings_Type] column)
Negotiated Savings =
CALCULATE([Total Savings], Savings[Savings_Type] = "Negotiated Price")

Volume Discount Savings =
CALCULATE([Total Savings], Savings[Savings_Type] = "Volume Discount")

Off-Contract Premium =
SUMX(
    FILTER(PO_Transactions, PO_Transactions[Is Compliant] = 0),
    PO_Transactions[PO_Amount] * 0.08
)
// ^ 8% premium is illustrative — adjust to your actual benchmark
```

---

### Visual 3 — Supplier Risk Matrix (Scatter Chart)

**Business Question:** *"Which suppliers combine HIGH spend exposure with LOW performance — creating the most strategic risk to the organization?"*

**Chart type:** Scatter plot

**X-axis:** `Total Spend` (financial exposure)
**Y-axis:** `Supplier Risk Score` (composite performance risk — higher = more risk)
**Bubble size:** `PO Count` (volume of transactions)
**Color:** Delivery Status (Green = >90% OTD, Amber = 80–90%, Red = <80%)

**Quadrant labels (add as text boxes):**
```
High Spend + High Risk → TOP RIGHT: "Critical — Immediate Action"
Low Spend + High Risk  → TOP LEFT:  "Watch List"
High Spend + Low Risk  → BOTTOM RIGHT: "Strategic Partners"
Low Spend + Low Risk   → BOTTOM LEFT:  "Standard Vendors"
```

**Measures used:**
- `Total Spend`
- `Supplier Risk Score` (calculated column from Page 2)
- `On Time Delivery Rate`
- `Total PO Count`

**Tooltip:** Supplier Name, Spend, OTD Rate, Invoice Accuracy, Contract Coverage %, Risk Score

**Interaction:** Click a bubble → drill-through to Page 2 filtered to that supplier.

---

### Visual 4 — Tail Spend Analysis (Donut + Bar)

**Business Question:** *"How much of our spend is fragmented, low-value, and difficult to manage? Is tail spend growing?"*

**Chart type:** Donut chart for proportion + Horizontal bar for category breakdown of tail spend

**Donut:**
- Segments: Strategic (≥$100K), Tactical ($25K–$99K), Operational ($5K–$24K), Tail Spend (<$5K)
- Measure: `Total Spend` by `Spend Tier`

**Bar (placed beside donut):**
- Y-axis: Category
- X-axis: Tail spend within each category
- Highlights categories with the most fragmented/unmanaged tail

**Measures used:**
```dax
Tail Spend Amount =
CALCULATE([Total Spend], PO_Transactions[Spend Tier] = "Tail Spend (<$5K)")

Tail Spend PO Count =
CALCULATE([Total PO Count], PO_Transactions[Spend Tier] = "Tail Spend (<$5K)")

Tail Spend Avg PO Value =
DIVIDE([Tail Spend Amount], [Tail Spend PO Count])

// Tail spend by category
Tail Spend by Category =
CALCULATE(
    [Total Spend],
    PO_Transactions[Spend Tier] = "Tail Spend (<$5K)"
)
```

**Why this matters:** If tail spend accounts for >20% of transactions but <5% of spend, consolidation opportunity exists. High tail spend in strategic categories signals process breakdowns.

---

---

## CROSS-PAGE INTERACTIONS & DRILL-THROUGH

| From | Action | Target |
|------|--------|--------|
| Page 1 — Supplier bar | Click supplier name | Filters Page 2 to that supplier |
| Page 1 — Category treemap | Click category | Filters all pages to that category |
| Page 2 — Supplier scorecard row | Right-click → Drill through | Page 3 filtered to that supplier's compliance/savings |
| Page 3 — Risk scatter bubble | Click | Drill-through to Page 2 supplier view |
| Any KPI Card | Click "Off-Contract Spend" | Navigate to Page 3 with non-compliant filter applied |

---

## REPORT-LEVEL SETTINGS

### Filters (report-level — apply across all pages)
- Year slicer (2022 / 2023 / 2024 / All)
- Quarter slicer
- Fiscal vs Calendar Year toggle (if applicable)

### Slicers (page-specific)
- Page 1: Category, Business Unit
- Page 2: Supplier, Contract Status (Active / Expiring / Expired)
- Page 3: Compliance Status (Compliant / Non-Compliant), Savings Type

### Bookmarks (recommended)
- "Executive View" — hides all drill-downs, shows KPIs + top-level charts
- "Operational View" — expands all matrices, shows row-level detail
- "Year-End Review" — pre-filters to full-year comparison

---

## COLOR PALETTE & THEME

```json
{
  "name": "Procurement Analytics",
  "dataColors": [
    "#1A56DB",  // Primary blue (spend, neutral)
    "#0E9F6E",  // Green (on-time, compliant, savings)
    "#E3A008",  // Amber (warning, near-threshold)
    "#E02424",  // Red (overbudget, non-compliant, risk)
    "#7E3AF2",  // Purple (contracts, secondary analysis)
    "#1C64F2",  // Light blue (PO volume)
    "#6B7280"   // Gray (baselines, prior year)
  ],
  "background": "#F9FAFB",
  "foreground": "#111827",
  "tableAccent": "#1A56DB"
}
```

---

## PERFORMANCE OPTIMIZATION NOTES

1. **Aggregation tables:** Pre-aggregate PO_Transactions by Month + Category + Supplier for trend visuals to reduce scan time on 5,200 rows (low impact here, but good practice).
2. **Disable auto date/time:** Turn off in File → Options → Data Load → disable "Auto date/time" to prevent hidden date tables.
3. **Incremental refresh:** If dataset grows beyond 50K rows, configure incremental refresh policies on PO_Date.
4. **Visual-level filters:** Apply filters at visual level (not page level) where possible to allow cross-filtering to work correctly.
5. **DirectQuery vs Import:** Import mode recommended for this dataset size (5,200 rows). Use DirectQuery only if near-real-time refresh is required.
