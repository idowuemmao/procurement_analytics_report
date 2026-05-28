# 📊 Procurement Analytics Report
### Power BI · 3-Page Executive Dashboard · 5,200 Records · 2022–2024

> *Transforming raw procurement data into decisions. Every visual earns its place.*

---

## 🗂 Table of Contents

- [Project Summary](#project-summary)
- [Screenshots](#screenshots)
- [Report Architecture](#report-architecture)
- [Page 1 — Spend Overview & Procurement Activity](#page-1--spend-overview--procurement-activity)
- [Page 2 — Supplier & Delivery Performance](#page-2--supplier--delivery-performance)
- [Page 3 — Savings, Risk & Compliance](#page-3--savings-risk--compliance)
- [Data Model & Calendar Table](#data-model--calendar-table)
- [Calculated Columns](#calculated-columns)
- [Full DAX Measure Library](#full-dax-measure-library)
- [Filter Architecture](#filter-architecture)
- [Key Findings](#key-findings)
- [Design Philosophy](#design-philosophy)
- [Tools & Technologies](#tools--technologies)
- [Targets Reference](#targets-reference)

---

## Project Summary

| | |
|---|---|
| **Tool** | Microsoft Power BI Desktop |
| **Dataset** | 5,200 procurement records — single flat fact table |
| **Period** | January 2022 – December 2024 |
| **Report pages** | 3 executive pages + drill-through layers |
| **KPI cards** | 18 across all pages |
| **DAX measures** | 40+ including dynamic insight text |
| **Calculated columns** | 6 |
| **Filter groups** | 6 per page · 30+ slicers |
| **Interactivity** | Cross-filtering · drill-down · drill-through · tooltips |
| **Primary spend metric** | `Line_Total_Inc_Tax` (final invoice value inc. tax) |

This report was built to answer one question before every design decision: *does this visual help a leader make a better decision?* Not to display data — to drive action.

The dataset covers purchasing transactions, supplier profiles, contract details, delivery records, invoice and payment data, budget allocations, and procurement savings — all unified in a single fact table linked to a custom Calendar table for full time intelligence.

---

## Screenshots

### Page 1 — Spend Overview & Procurement Activity

<img width="2460" height="1410" alt="1" src="https://github.com/user-attachments/assets/b495e4ad-3797-4d41-957d-f2570dead737" />

> Total spend of £979.40M across 5,200 PO lines. Monthly YoY trend with green/red bar combo chart. Category and item-level drill-down. Emergency PO exposure flagged at 10.6% against a 5% target.

### Page 2 — Supplier & Delivery Performance

<img width="2460" height="1410" alt="2" src="https://github.com/user-attachments/assets/9ddf5d85-6988-4012-baf4-b510e99b8c34" />

> OTIF rate of 64.13% against a 95% target. Scatter plot exposing spend vs delivery correlation. Global supplier map. Full supplier scorecard with conditional formatting across 9 performance dimensions.

### Page 3 — Savings, Risk & Compliance

<img width="2460" height="1410" alt="3" src="https://github.com/user-attachments/assets/bf4ec848-e260-4d39-acbd-cab3966d8610" />

> £104.27M savings at 10.29% rate. Maverick spend of £105.40M breaching the zero target. Invoice matching gap — only 41.13% 3-way matched. Department-level compliance accountability chart.

---

## Report Architecture

The three pages follow a deliberate narrative arc. Each page closes one question and opens the next.

```
┌─────────────────────────────────────────────────────────────────┐
│  Page 1 — SPEND OVERVIEW & PROCUREMENT ACTIVITY                 │
│  How much are we spending, where is it going, and how is        │
│  it trending over time?                                         │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ▼  Now we know the scale of spend —
                                who is responsible for delivering it?
┌─────────────────────────────────────────────────────────────────┐
│  Page 2 — SUPPLIER & DELIVERY PERFORMANCE                       │
│  Which suppliers are failing us, and what is the delivery       │
│  and lead time risk across our supply chain?                    │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ▼  We know where money goes and who
                                delivers it — are controls holding?
┌─────────────────────────────────────────────────────────────────┐
│  Page 3 — SAVINGS, RISK & COMPLIANCE                            │
│  Are we capturing the savings we should, and where are the      │
│  financial control failures exposing the organisation?          │
└─────────────────────────────────────────────────────────────────┘
```

---

## Page 1 — Spend Overview & Procurement Activity

**Executive question:** *Where is money going and is our spending under control across categories, departments, and time?*

### KPI Cards

| KPI | Value | Insight |
|---|---|---|
| **Total Spend** | £979.40M | 50.2% YoY growth. Below budget. Avg tax 8.2% · Total tax £70.6M |
| **Total PO Lines** | 5,200 | Orders raised above benchmark, suggesting consolidated purchasing |
| **Avg PO Value** | £188.35K | Elevated — Emergency PO rate (10%) may be inflating unit costs |
| **Budget Variance %** | 3.32% | Below budget by £33.7M — within expected spending range |
| **Emergency PO %** | 10.17% (529 POs) | £108.8M emergency spend — above target. Review root causes by department |
| **Cancelled PO %** | 10.60% | £115.7M cancellation spend — signals demand forecasting or approval failure |

Each KPI card includes a dynamic insight text measure that updates with filter context. Micro sparkline bars show the three-year trend inline.

### Visuals

**V1 — How is monthly spending trending year on year?**
*Combo chart: clustered columns (YoY % change per month) with drill-down to department*

The centrepiece of Page 1. Dark navy bars show months of year-on-year growth; pink bars show decline. The YoY % label sits above or below each bar. Drilling down to department level reveals which business units are driving spend acceleration or reduction in any given month. Subtitle: *2023 vs 2022 and 2024 vs 2023 — identifies whether spend growth is accelerating, stabilising, or reversing across the procurement cycle.*

**V2 — Where is our spend and volume concentrated?**
*Bar chart with drill-down to subcategory*

Sorted descending by spend. IT Software leads at £323.80M. A line overlay shows PO count per category, separating volume-driven from price-driven categories. Drill to subcategory without leaving the visual.

**V3 — Spend distribution by PO type**
*Donut chart with drill-down to PO status*

Centre value: £1.01bn total budget. Standard POs = £568.36M (58%). Emergency = £108.75M (11.1%). Blanket = £201.99M (20.6%). Contract = £100.29M (10.2%). Each segment drills to PO status (Open / Closed / Cancelled / Disputed) to surface unresolved spend within each purchasing route.

**V4 — Which items are driving spend, savings, and risk — and where do we need to act?**
*Matrix table with conditional formatting*

An operational item-level view for procurement managers. Columns: Item Description · Total Spend · Budget Variance % · Emergency PO % · PO Count · Total Savings · Savings Rate · Avg PO Value · On Time Delivery %. Conditional formatting on Budget Variance % (red = overspend) and On Time Delivery % (blue gradient — darker = better).

---

## Page 2 — Supplier & Delivery Performance

**Executive question:** *Which suppliers are failing us, and what is the delivery and lead time risk across our supply chain?*

### KPI Cards

| KPI | Value | Insight |
|---|---|---|
| **On Time Delivery** | 3,335 · 64.13% | Delivered on time 64.13% of the time — ▲ up vs last year; target is 95% |
| **Avg Lead Time** | 36 days | Ranging 0 to 90 days — lead times extended, review slow categories |
| **Avg Days Late** | 16 days | Each day late increases downstream operational cost |
| **Tier 1 Spend** | £326.90M · 33.38% | Consider consolidating spend toward strategic partners |
| **High Risk Spend** | £63.52M · 6.49% | Within tolerance but requires active monitoring |
| **Single Source** | £122.00M · 11.87% | Monitor for any new sole-source contracts entering the pipeline |

### Visuals

**V1 — Spend vs on-time delivery by supplier**
*Scatter plot*

X-axis: On Time Delivery %. Y-axis: Total Spend. Colour by supplier preference (Preferred · Strategic · Transactional). Average reference lines on both axes create four quadrants. The top-left danger zone (high spend, low OTIF) is where procurement action is most urgent. Subtitle: *Does higher spend with a supplier guarantee better delivery, or are we paying more for poor performance?*

**V2 — Spend and delivery performance by supplier region → country**
*Bar and line combo with drill-down*

Bar height = Total Spend by region. Line = On Time Delivery % for same region. Europe leads at £454.94M with 65.46% OTIF. Oceania has the lowest spend (£79.26M) but shows 66.57% OTIF — still far short of the 95% target. Drill to country level to pinpoint delivery failure geographically.

**V3 — Spend concentration by supplier tier → PO status**
*Donut chart with drill-down*

Centre value: 15 total suppliers. Strategic = £326.90M (33.38%), Preferred = £476.63M (48.67%), Transactional = £175.87M (17.96%). Each tier shows a YoY variance indicator. Drill to PO status within each tier.

**V4 — Global supplier spend and risk concentration**
*Map visual using Supplier_Latitude and Supplier_Longitude*

Bubble size proportional to spend. Colour indicates supplier tier. Europe is the dominant cluster (£454.94M). Immediate visibility of single-region dependency and geographic delivery risk. Tooltip shows supplier name, tier, risk level, OTIF %, and spend on hover.

**V5 — Supplier scorecard — full performance matrix**
*Table with conditional formatting*

The centrepiece of Page 2. Every active supplier in a single view. Columns: Supplier Name · Tier (badge) · Risk (badge) · Total Spend · Budget Variance % · PO Count · Avg Lead Time · Avg Days Late · On Time Delivery % · Avg ESG Score · Preference. Conditional formatting: OTIF % uses a blue gradient, Budget Variance % turns red for negative values. Sortable by any column. Drill-through to a supplier detail page showing all PO lines for that supplier.

Sample visible in report:

| Supplier | Tier | Risk | Total Spend | Bud Var | OTIF % | ESG |
|---|---|---|---|---|---|---|
| Apex Industrial Supplies | Strategic | Low | £60.37M | +2.83% | 62.81% | 75.20 |
| Atlantic Raw Materials | Transactional | High | £63.52M | -0.26% | 63.14% | 52.00 |
| Blue Horizon | Preferred | Low | £79.26M | +5.85% | 66.57% | 50.90 |

---

## Page 3 — Savings, Risk & Compliance

**Executive question:** *Are we capturing the savings we should, and where are the financial control failures that expose the organisation?*

### KPI Cards

| KPI | Value | Insight |
|---|---|---|
| **Compliant Spend** | £874.00M · 89.24% | ⚠ 0.8% below target. £105.4M outside approved channels. Primary driver: Procurement |
| **Total Saving** | £104.27M · 10.29% | Within target range (10–20%). Monitor for consistency |
| **Maverick Spend** | £105.40M · 10.76% | ⛔ Target breached. Procurement at 14.5%. Escalate immediately |
| **3-Way Match** | 2,139 · 41.13% | Below the 80%+ gold standard for payables control |
| **Overdue Invoice** | £136.37M · 15.52% | Requires active monitoring — payment relationship risk |
| **Overspend Amount** | -£18.66M | Category-level investigation required |

Dynamic insight text measures drive the sub-text beneath each KPI. The maverick spend card switches between four states (zero / monitor / warning / critical) based on the current filtered value and always names the worst offending department.

### Visuals

**V1 — Savings delivery over time — are we on track and is the savings rate improving?**
*Combo chart: columns + line · quarterly drill*

Columns show monthly savings value — dark navy for positive months, pink for overspend (January 2023: -£5.54M). Line shows savings rate % on the same axis. January 2024 peaks at 12.53%. The quarterly drill removes month-level noise and shows directional trend more clearly.

**V2 — Savings distribution by invoice aging band**
*Donut chart*

Segments: Paid (32.10% · +£33.47M) and Overdue 90+ days (67.90% · -£618.23M). Centre KPI: £979.40M total spend. The critical finding: overdue 90+ day invoices are responsible for -£618.23M in savings destruction — the single largest financial risk finding in the report. Labels show savings efficiency within each band, not savings as a share of total savings.

**V3 — Does invoice matching quality affect the spend we can control?**
*Clustered column*

X-axis: Invoice Match Type (3-Way · 2-Way · No Match). Legend: Invoice Status (Disputed · Overdue · Paid · Pending). 3-Way Match carries the most Paid spend (£182.42M). No Match cluster carries £168.35M of Paid spend — payments made without a full audit trail. The 2-Way cluster shows the highest Overdue volume proportionally.

**V4 — Which categories are generating savings — and which are consistently overspending?**
*Stacked bar with drill-down to subcategory*

Dark navy = Total Savings. Pink = Overspend Amount. Sorted by savings descending. IT Software: £35.91M savings, -£6.26M overspend. Professional Services: £20.63M savings, -£2.49M overspend. Raw Materials: £46.36K savings, -£4.88K overspend. Drill to subcategory to identify specific items driving budget leakage.

**V5 — Which departments are most exposed to non-compliant spend?**
*Clustered column + line*

Grey bars = compliant spend per department. Red line = maverick spend value overlay. Marketing leads on maverick spend at £17.74M against £126.72M compliant spend. Legal has the highest maverick ratio relative to its spend volume. Sorted by maverick spend descending for immediate accountability visibility.

---

## Data Model & Calendar Table

```
data (fact table — 5,200 rows)
  └── data[PO_Date]  →  Calendar[Date]  (many-to-one · active relationship)
```

All dimensions (supplier, department, category, contract, invoice, payment) are embedded in the single fact table. No snowflake structure. The Calendar table is marked as the Date Table in Model view, enabling all time intelligence functions.

```dax
Calendar =
ADDCOLUMNS(
    CALENDAR(DATE(2022, 1, 1), DATE(2024, 12, 31)),
    "Year",           YEAR([Date]),
    "Quarter",        "Q" & QUARTER([Date]),
    "Month Number",   MONTH([Date]),
    "Month Name",     FORMAT([Date], "MMMM"),
    "Month Short",    FORMAT([Date], "MMM"),
    "Week Number",    WEEKNUM([Date]),
    "Fiscal Year",    IF(MONTH([Date]) >= 4,
                          "FY" & YEAR([Date]) & "/" & RIGHT(YEAR([Date]) + 1, 2),
                          "FY" & YEAR([Date]) - 1 & "/" & RIGHT(YEAR([Date]), 2)),
    "Fiscal Quarter", IF(MONTH([Date]) >= 4,
                          "Q" & INT((MONTH([Date]) - 4) / 3) + 1,
                          "Q" & INT((MONTH([Date]) + 8) / 3) + 1),
    "Is Weekend",     IF(WEEKDAY([Date], 2) >= 6, TRUE, FALSE)
)
```

**Key field reference:**

| Field | Role | Notes |
|---|---|---|
| `Line_Total_Inc_Tax` | Primary spend metric | Final invoice value including tax |
| `Line_Net` | Secondary spend metric | Ex-tax analysis |
| `Savings_Amount` | Savings / overspend | Positive = saving · Negative = overspend |
| `Supplier_Latitude` / `Supplier_Longitude` | Map visual | Country centroid coordinates |
| `On_Time_Delivery` | OTIF KPI driver | Yes / No · target ≥ 95% |
| `Maverick_Spend` | Compliance flag | Yes / No · target = 0% |
| `Invoice_Match_Type` | Control quality | 3-Way / 2-Way / No Match |
| `Supplier_Tier` | Sourcing strategy | 1 = Strategic · 2 = Preferred · 3 = Transactional |

---

## Calculated Columns

All columns added to the `data` table.

```dax
-- 1. Numeric risk score for scatter plot axis and sorting
Risk_Score_Num =
SWITCH(data[Supplier_Risk],
    "Low",    1,
    "Medium", 2,
    "High",   3,
    0)

-- 2. Delivery status band for conditional formatting and filtering
Delivery_Status =
SWITCH(TRUE(),
    data[Days_Late] <= 0, "On Time",
    data[Days_Late] <= 3, "Slightly Late",
    data[Days_Late] <= 7, "Moderately Late",
    "Severely Late")

-- 3. Savings band for grouping in distribution charts
Savings_Band =
SWITCH(TRUE(),
    data[Savings_Pct] >= 15, "High Saving ≥15%",
    data[Savings_Pct] >= 5,  "Saving 5–15%",
    data[Savings_Pct] >= 0,  "Minimal Saving 0–5%",
    data[Savings_Pct] < 0,   "Overspend")

-- 4. PO age bucket for aging analysis
PO_Age_Bucket =
VAR AgeDays = DATEDIFF(data[PO_Date], TODAY(), DAY)
RETURN
SWITCH(TRUE(),
    AgeDays <= 30,  "0–30 days",
    AgeDays <= 90,  "31–90 days",
    AgeDays <= 180, "91–180 days",
    "> 180 days")

-- 5. Contract flag — separates contracted from spot purchases
Has_Contract =
IF(data[Contract_ID] = "NO-CONTRACT", "No Contract", "Contracted")

-- 6. Fiscal year label derived from PO date (UK fiscal: April start)
Fiscal_Year_Label =
VAR m = MONTH(data[PO_Date])
VAR y = YEAR(data[PO_Date])
RETURN
IF(m >= 4,
    "FY" & y & "/" & RIGHT(y + 1, 2),
    "FY" & (y - 1) & "/" & RIGHT(y, 2))
```

---

## Full DAX Measure Library

### Core spend

```dax
Total Spend =
SUM(data[Line_Total_Inc_Tax])

Total Spend Net =
SUM(data[Line_Net])

PO Count =
COUNTROWS(data)

Avg PO Value =
DIVIDE([Total Spend], [PO Count])
```

### Time intelligence

```dax
Spend PY =
CALCULATE(
    [Total Spend],
    SAMEPERIODLASTYEAR('Calendar'[Date])
)

Spend YoY % Change =
DIVIDE(
    [Total Spend] - [Spend PY],
    [Spend PY],
    BLANK()          -- returns BLANK not error when PY = 0
)

Spend YoY Abs Change =
[Total Spend] - [Spend PY]

Spend YoY Direction =
IF([Spend YoY % Change] >= 0, 1, -1)

Spend 3M Rolling Avg =
CALCULATE(
    AVERAGEX(
        DATESINPERIOD(
            'Calendar'[Date],
            LASTDATE('Calendar'[Date]),
            -3, MONTH
        ),
        [Total Spend]
    )
)

Spend YTD =
TOTALYTD(
    [Total Spend],
    'Calendar'[Date],
    "31/03"          -- UK fiscal year end
)
```

### Budget & savings

```dax
Total Budget =
SUM(data[Budget_Total])

Budget Variance =
[Total Spend] - [Total Budget]

Budget Variance % =
DIVIDE([Budget Variance], [Total Budget])

Total Savings =
CALCULATE(
    SUM(data[Savings_Amount]),
    data[Savings_Amount] > 0
)

Total Overspend =
CALCULATE(
    SUM(data[Savings_Amount]),
    data[Savings_Amount] < 0
)

Savings Rate % =
DIVIDE([Total Savings], [Total Budget])
```

### Supplier & delivery

```dax
OTIF % =
DIVIDE(
    CALCULATE([PO Count], data[On_Time_Delivery] = "Yes"),
    [PO Count]
)

OTIF Target = 0.95

OTIF vs Target =
[OTIF %] - [OTIF Target]

Avg Lead Time =
AVERAGE(data[Lead_Time_Days])

Avg Days Late =
CALCULATE(
    AVERAGE(data[Days_Late]),
    data[Days_Late] > 0
)

Late Delivery Count =
CALCULATE([PO Count], data[On_Time_Delivery] = "No")

Avg ESG Score =
AVERAGE(data[Supplier_ESG_Score])

Tier 1 Spend =
CALCULATE([Total Spend], data[Supplier_Tier] = 1)

Tier 1 Spend % =
DIVIDE([Tier 1 Spend], [Total Spend])

High Risk Spend =
CALCULATE([Total Spend], data[Supplier_Risk] = "High")

Single Source Spend =
CALCULATE([Total Spend], data[Single_Source_Flag] = "Yes")

Single Source % =
DIVIDE([Single Source Spend], [Total Spend])
```

### Compliance & financial control

```dax
Compliant Spend Value =
CALCULATE([Total Spend], data[Maverick_Spend] = "No")

Compliant Spend % =
DIVIDE([Compliant Spend Value], [Total Spend])

Maverick Spend Value =
CALCULATE([Total Spend], data[Maverick_Spend] = "Yes")

Maverick Spend % =
DIVIDE([Maverick Spend Value], [Total Spend])

3-Way Match % =
DIVIDE(
    CALCULATE([PO Count], data[Invoice_Match_Type] = "3-Way Match"),
    [PO Count]
)

No Match Value =
CALCULATE([Total Spend], data[Invoice_Match_Type] = "No Match")

Match Risk Spend =
CALCULATE(
    [Total Spend],
    data[Invoice_Match_Type] IN {"No Match", "2-Way Match"}
)

Overdue Invoice Value =
CALCULATE([Total Spend], data[Invoice_Status] = "Overdue")

Payment Risk Exposure =
CALCULATE(
    [Total Spend],
    data[Payment_Status] IN {"Overdue", "On Hold"}
)

Emergency PO % =
DIVIDE(
    CALCULATE([PO Count], data[PO_Type] = "Emergency"),
    [PO Count]
)

Cancelled PO % =
DIVIDE(
    CALCULATE([PO Count], data[PO_Status] = "Cancelled"),
    [PO Count]
)
```

### Dynamic insight text

```dax
Compliant Spend Insight Text =
VAR _pct       = [Compliant Spend %]
VAR _mavVal    = FORMAT([Maverick Spend Value] / 1000000, "0.0m")
VAR _mavPct    = FORMAT(1 - _pct, "0.0%")
VAR _pctFmt    = FORMAT(_pct, "0.0%")
VAR _gap       = FORMAT(ABS(_pct - 0.90), "0.0%")
VAR _worstDept =
    CALCULATE(
        FIRSTNONBLANK(data[Department], 1),
        TOPN(1, VALUES(data[Department]), [Maverick Spend %], DESC)
    )
RETURN
SWITCH(TRUE(),
    _pct >= 0.90,
        "✓ " & _gap & " above target. Maverick spend: " & _mavVal &
        " (" & _mavPct & "). Lowest compliance department: " & _worstDept & ".",
    _pct >= 0.80,
        "⚠ " & _gap & " below target. " & _mavVal &
        " committed outside approved channels. Primary driver: " & _worstDept & ".",
        "⛔ " & _pctFmt & " — " & _gap &
        " below floor. " & _mavVal &
        " outside policy. Immediate escalation required. Worst department: " & _worstDept & "."
)

Maverick Spend Insight Text =
VAR _pct       = [Maverick Spend %]
VAR _val       = FORMAT([Maverick Spend Value] / 1000000, "£0.0M")
VAR _pctFmt    = FORMAT(_pct, "0.0%")
VAR _worstDept =
    CALCULATE(
        FIRSTNONBLANK(data[Department], 1),
        TOPN(1, VALUES(data[Department]), [Maverick Spend %], DESC)
    )
VAR _worstPct  = FORMAT(
    CALCULATE([Maverick Spend %], data[Department] = _worstDept),
    "0.0%")
RETURN
SWITCH(TRUE(),
    _pct = 0,
        "✓ No maverick spend recorded this period. All purchases passed through approved channels.",
    _pct <= 0.05,
        "⚠ " & _val & " (" & _pctFmt & ") outside policy — within tolerance but monitor " &
        _worstDept & " (" & _worstPct & ").",
    _pct <= 0.10,
        "⚠ " & _val & " (" & _pctFmt & ") bypassed procurement. " & _worstDept &
        " is the primary driver at " & _worstPct & ". Review pre-approval controls.",
        "⛔ " & _val & " (" & _pctFmt & ") outside policy — target breached. " &
        _worstDept & " at " & _worstPct & ". Escalate immediately."
)
```

### Filter intelligence

```dax
-- Counts active slicer selections per filter group.
-- Drop into a Card visual beside each filter group header.

Active Supplier Filters =
VAR _name    = ISFILTERED(data[Supplier_Name])
VAR _risk    = ISFILTERED(data[Supplier_Risk])
VAR _tier    = ISFILTERED(data[Supplier_Tier])
VAR _pref    = ISFILTERED(data[Preferred_Supplier])
VAR _status  = ISFILTERED(data[Supplier_Status])
RETURN
CONVERT(_name, INTEGER) +
CONVERT(_risk, INTEGER) +
CONVERT(_tier, INTEGER) +
CONVERT(_pref, INTEGER) +
CONVERT(_status, INTEGER)
```

---

## Filter Architecture

Each page uses a structured filter pane that opens from the **OPEN FILTER** button. Slicers are grouped into labelled sections. A filter intelligence counter shows active selections in each group — so users always know what context they are viewing. The filter bar across the top of each page displays `Selected Slicers: GroupName(n) | GroupName(n) | ...`

### Page 1 — Supplier, Geography, Contract, Invoice, Payment, Risk

| Group | Slicers |
|---|---|
| Supplier | Supplier Name · Supplier Risk · Supplier Tier · Supplier Preference · Supplier Status |
| Geography | Supplier Region · Supplier Country |
| Contract | Requestor Name · Approver Name · Contract Type · Match Quality · Contract Duration |
| Invoice | Invoice Match Type · Invoice Status · Invoice Age Band · Delivery Status |
| Payment | Currency · Unit of Measure · Payment Terms · Payment Status |
| Risk | Maverick Spend · Single Source Flag · Sourcing Risk Flag |

### Page 2 — Calendar, Procurement, Contract, Invoice, Payment, Risk

| Group | Slicers |
|---|---|
| Calendar | Date Between · Date Hierarchy · Tax Percentage · Day Name · Item Description |
| Procurement | Department · Category · Subcategory · PO Status · PO Type |
| Contract | Requestor Name · Approver Name · Contract Type · Match Quality · Contract Duration |
| Invoice | Invoice Match Type · Invoice Status · Invoice Age Band · Delivery Status |
| Payment | Currency · Unit of Measure · Payment Terms · Payment Status |
| Risk | Maverick Spend · Single Source Flag · Sourcing Risk Flag |

### Page 3 — Supplier, Geography, Contract, Operations, Payment, Risk

| Group | Slicers |
|---|---|
| Supplier | Supplier Name · Supplier Risk · Supplier Tier · Supplier Preference · Supplier Status |
| Geography | Supplier Region · Supplier Country |
| Contract | Requestor Name · Approver Name · Contract Type · Match Quality · Contract Duration |
| Operations | Date Hierarchy · Item Description · PO Status · PO Type · Delivery Status |
| Payment | Currency · Unit of Measure · Payment Terms · Payment Status |
| Risk | Maverick Spend · Single Source Flag · Sourcing Risk Flag |

---

## Key Findings

All values in local currency — no FX conversion applied.

| Area | Finding | Value | Status |
|---|---|---|---|
| Spend | Total procurement spend 2022–2024 | £979.40M | — |
| Spend | Year-on-year growth | +50.2% | ▲ Strong |
| Spend | Budget variance | -3.32% | ✓ Within range |
| Spend | Emergency PO rate | 10.60% | ✗ Target < 5% |
| Spend | Cancelled PO rate | 10.17% | ⚠ Monitor |
| Delivery | On-time delivery rate | 64.13% | ✗ Target ≥ 95% |
| Delivery | Average lead time | 36 days | ⚠ Range 0–90 days |
| Delivery | Average days late | 16 days | ⚠ Review categories |
| Supplier | Tier 1 (Strategic) spend share | 33.38% | ⚠ Consolidate |
| Supplier | High-risk supplier spend | £63.52M · 6.49% | ⚠ Monitor |
| Supplier | Single-source spend | £122.00M · 11.87% | ⚠ Diversify |
| Savings | Total savings delivered | £104.27M · 10.29% | ✓ Target 10–20% |
| Savings | Total overspend | -£18.66M | ⚠ Category review |
| Compliance | Compliant spend | £874.00M · 89.24% | ⚠ Target ≥ 90% |
| Compliance | Maverick spend | £105.40M · 10.76% | ✗ Target = 0% |
| Control | 3-Way invoice match rate | 41.13% | ✗ Below standard |
| Control | Overdue invoices | £136.37M · 15.52% | ⚠ Monitor |
| Control | Overdue 90+ day savings impact | -£618.23M | ✗ Critical risk |

---

## Design Philosophy

**Every visual must earn its place.** Before any chart was added to the report, it was tested against one question: *does this help a leader make a decision they could not otherwise make?* Visuals that describe data without enabling action were excluded.

**The report is a narrative, not a dashboard.** The three-page structure builds a case: establish the financial scale (Page 1), diagnose operational execution (Page 2), confirm whether controls are holding (Page 3). A reader who moves through all three pages arrives at a complete picture of procurement health without jumping between pages to form a view.

**The KPI card is a sentence, not a number.** Dynamic insight text measures ensure every KPI card communicates a value, a gap to target, the primary driver, and a recommended action — in two lines. The report speaks to the reader rather than making them interpret raw figures.

**Filter transparency prevents misreading.** Filter intelligence counters in the filter bar mean consumers always know what slice of data they are viewing. A clean filter bar with zero counts is an explicit confirmation the full dataset is in scope.

**Conditional formatting is deliberate, not decorative.** Colour communicates status: blue gradient for OTIF (darker = better), red for negative budget variance, green/red for savings/overspend columns. Colour is never used purely for visual variety.

**Drill-down is earned, not assumed.** Every visual that supports drill-down signals it in the subtitle (e.g. *DRILL DOWN TO SUBCATEGORY*) so users know the interaction is available without trial and error.

---

## Tools & Technologies

| | |
|---|---|
| **Reporting** | Microsoft Power BI Desktop |
| **Query language** | DAX (Data Analysis Expressions) |
| **Data preparation** | Power Query (M language) |
| **Map visual** | Bing Maps (lat/long coordinates) |
| **Time intelligence** | Custom Calendar table · marked as Date Table |
| **Dynamic text** | DAX string measures in Card visuals |
| **Conditional formatting** | Measure-driven rules across all three pages |

---

## Targets Reference

| Metric | Target | Rationale |
|---|---|---|
| OTIF % | Reliability % | Industry standard for supply chain reliability |
| Maverick Spend % | 0% | Any bypass is a policy breach — no tolerance |
| Compliant Spend % | ≥ 90% | Practical floor accounting for approved exceptions |
| Savings Rate % | 10–20% | Balanced target: efficiency without under-investment |
| Emergency PO % | < 5% | Higher rates signal planning and process failure |
| 3-Way Match % | > 80% | Financial control standard for payables integrity |

---
