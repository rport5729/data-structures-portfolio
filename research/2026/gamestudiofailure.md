# Game Publishers Finanical Data Research Study - THQ
---

## 1. Problem Definition

## Research Question: How can financial data from consolidated income statements be used to identify and monitor the stages of a game publisher's failure?


### Context
This case study examines **THQ** (creators of Saints Row and Darksiders franchises), a major game publisher that declined between 2010-2012. Understanding the  indicators that signal failure is import[...]

### Relevance
Game publishers face pressure that can be examined and applied to businesses in other genres to help them avoid the same failures that the game publishers go through. Revenue from products that depend[...]

---

## 2. Data Description

### Key Variables and Operationalization

| Variable | Definition | Measurement | Unit |
|----------|-----------|-------------|------|
| **Revenue** | Total sales from product sales | Annual reported revenue | $K (thousands) |
| **Product Costs** | Direct manufacturing/distribution costs | COGS (Cost of Goods Sold) line item | $K |
| **Software Amortization & Royalties** | Capitalized software development written down over time + licensing fees | COGS line item | $K |
| **License Amortization & Royalties** | IP licensing and franchise royalty costs | COGS line item | $K |
| **Total COGS** | Sum of all product-related costs | Product Costs + Software + License Amortization | $K |
| **Gross Profit** | Revenue minus COGS | Revenue - Total COGS | $K |
| **Gross Margin %** | Profitability after direct costs | (Gross Profit / Revenue) × 100 | Percentage |
| **R&D Expense** | Research and development spending | Operating Expense line item | $K |
| **Selling Expense** | Marketing and distribution spending | Operating Expense line item | $K |
| **G&A Expense** | General and administrative overhead | Operating Expense line item | $K |
| **Restructuring Costs** | One-time layoff, reorganization, facility closure costs | Operating Expense line item | $K |
| **Total Operating Expenses** | Sum of R&D + Selling + G&A + Restructuring | SUM(all OpEx categories) | $K |
| **Operating Income** | Gross Profit minus Operating Expenses | Gross Profit - Total OpEx | $K |
| **Operating Margin %** | Operating profitability as % of revenue | (Operating Income / Revenue) × 100 | Percentage |
| **Net Income** | Bottom-line profit after all expenses | Reported net income | $K |
| **EPS (Basic)** | Earnings per share on basic shares | Net Income / Basic Shares Outstanding | $ per share |
| **EPS (Diluted)** | Earnings per share including potential dilution | Net Income / Diluted Shares Outstanding | $ per share |
| **Shares Outstanding** | Number of common shares | Reported shares outstanding | K (thousands) |
| **COGS as % of Revenue** | Cost structure intensity | (Total COGS / Revenue) × 100 | Percentage |

### Data Source
- **Primary Source:** THQ Inc. Consolidated Statements of Operations
- **Time Period:** Fiscal years ending March 31, 2010; March 31, 2011; March 31, 2012
- **Filing Type:** Annual 10-K SEC filings
- **Citation Format:** THQ Inc., Form 10-K Annual Reports, filed with the SEC

**Data Retrieval Method:**

Financial data was accessed programmatically from the SEC EDGAR API using the `edgar-py` Python library. The code below retrieves THQ's consolidated financial statements:

```python
from edgar import Company, set_identity

# Set identity with email for SEC EDGAR API access
set_identity("rport5729@gmail.com")

# THQ Inc. SEC CIK: 865570
company = Company("865570")

print(company.name)
print(company.cik)

# Retrieve financial data from 10-K filings
financials = company.get_financials()

# Extract income statement data (contains Revenue, COGS, Operating Expenses, Net Income)
income_statement = financials.income_statement()

# Extract balance sheet data (contains Shares Outstanding)
balance_sheet = financials.balance_sheet()

# Process and consolidate data for fiscal years ending March 31, 2010-2012
```

**Alternative Access Method:** SEC EDGAR filings can also be accessed directly at https://www.sec.gov/cgi-bin/browse-edgar?action=getcompany&CIK=0000865570&type=10-K&dateb=&owner=exclude&count=100

### Data Structure
- **Row Unit:** Each row represents a **fiscal year** (annual financial snapshot)
- **Time Resolution:** Annual data (fiscal years ending March 31)
- **Main Features:** 
  - Revenue and COGS breakdown by category (Products, Software Amortization, License Amortization)
  - Operating expense breakdown (R&D, Selling, G&A, Restructuring)
  - Calculated profitability metrics (Gross Profit, Operating Income, Net Income)
  - Per-share metrics (EPS and shares outstanding)
  - Derived financial ratios (margins, cost ratios)

### Dataset Size and Assumptions
- **Number of Observations:** 3 fiscal years (2010, 2011, 2012)
- **Number of Variables:** 17+ financial metrics
- **Completeness:** All reported line items from audited financial statements
- **Assumptions:**
  - Data is from audited SEC 10-K filings (high reliability)
  - Currency is USD in thousands ($K)
  - Fiscal year ends March 31 (consistent across all three years)
  - All figures represent actual reported values; no pro forma or adjusted earnings
  - No restatements or corrections applied post-filing

---

## 3. Data Cleaning and Preparation

### Step 1: Data Aggregation and Organization
**Action:** Consolidated individual line items from three separate 10-K filings into unified tables.

**Rationale:** SEC filings present data in different formats. Consolidating into consistent rows/columns enables comparison and analysis.

### Step 2: Variable Calculation
**Variables Calculated:**

| Calculation | Formula | Purpose |
|-------------|---------|---------|
| Gross Profit | `Revenue - Total_COGS` | Measure profitability after direct costs |
| Gross Margin % | `(Gross_Profit / Revenue) × 100` | Normalize gross profit as % of revenue |
| Operating Margin % | `(Operating_Income / Revenue) × 100` | Normalize operating profit as % of revenue |
| COGS as % of Revenue | `(Total_COGS / Revenue) × 100` | Measure cost structure intensity |
| OpEx as % of Revenue | `(Total_OpEx / Revenue) × 100` | Measure operating efficiency |
| EPS Change YoY | `EPS_Basic.pct_change() × 100` | Track year-over-year earnings trend |

**Rationale:** Percentage-based metrics normalize across years and allow for trend identification independent of absolute revenue changes.

### Step 3: Derived Failure Stage Indicators
**Variables Created:**

| Indicator | Calculation | Warning Sign |
|-----------|-------------|--------------|
| OpEx Revenue Ratio | `Total_OpEx / Revenue` | Identifies when OpEx grows faster than revenue |
| Restructuring Intensity | `(Restructuring_Costs / Revenue) × 100` | Flags organizational distress and layoffs |
| COGS Revenue Ratio | `Total_COGS / Revenue` | Tracks cost of sales efficiency erosion |
| Gross Margin Decline | `Gross_Margin_Pct.diff()` | Measures deterioration in profitability |
| Operating Loss Acceleration | `Operating_Income.diff()` | Detects worsening operating performance |

**Rationale:** These composite indicators flag warning signs: when COGS + OpEx exceed 100% of revenue, a company is spending more than it earns and cannot be sustained.

### Step 4: Data Validation
**Checks Performed:**

| Validation Check | Purpose |
|------------------|---------|
| `Revenue - Total_COGS == Gross_Profit` | Verify accounting identity for gross profit |
| `Gross_Profit - Total_OpEx == Operating_Income` | Verify accounting identity for operating income |
| No null values in: Revenue, Total_COGS, Operating_Income | Ensure data completeness in critical fields |
| `Total_COGS > 0` | Verify COGS is logically positive |
| `Revenue > 0` | Verify revenue is logically positive |

**Rationale:** Financial data must satisfy accounting identities; validating these ensures data integrity before analysis.

### Step 5: No Data Removed
**Decision:** All three fiscal years retained (no rows deleted).

**Rationale:** 
- Dataset is small (3 observations), and removing any year would eliminate critical historical context
- Data quality is high (audited SEC filings)
- The year with worst performance (2012) is essential to understanding failure progression

### Step 6: No Missing Value Handling Required
**Observation:** All reported financial figures are present in original 10-K filings.

**Rationale:** SEC-mandated consolidated statements include all line items; missing values would indicate incomplete filing or data transcription error.

---

## Raw Data Summary

### Revenue Trends

| Fiscal Year | Revenue | Change |
|-------------|---------|--------|
| Mar 31, 2010 | $899,137K | - |
| Mar 31, 2011 | $665,258K | -26.0% ↓ |
| Mar 31, 2012 | $830,841K | +24.9% ↑ |

### Cost of Sales Breakdown

| Component | 2010 | 2011 | 2012 |
|-----------|------|------|------|
| **Product Costs** | $318,590K | $272,021K | $353,597K |
| **Software Amortization & Royalties** | $196,956K | $129,237K | $308,051K |
| **License Amortization & Royalties** | $110,503K | $118,287K | $74,632K |
| **Total COGS** | $626,049K | $519,545K | $736,280K |

### Profitability Analysis

| Metric | 2010 | 2011 | 2012 |
|--------|------|------|------|
| Gross Profit | $273,088K | $145,713K | $94,561K |
| Gross Margin % | 30.4% | 21.9% | 11.4% |
| Operating Income | $(9,649)K | $(135,694)K | $(242,149)K |
| Net Income | $(9,017)K | $(136,098)K | $(242,506)K |

### Operating Expenses

| Category | 2010 | 2011 | 2012 |
|----------|------|------|------|
| R&D | $87,233K | $79,374K | $89,526K |
| Selling | $131,954K | $156,075K | $191,669K |
| G&A | $57,879K | $45,356K | $48,712K |
| Restructuring | $5,671K | $602K | $6,803K |
| **Total OpEx** | $282,737K | $281,407K | $336,710K |

### Performance Indicators

| KPI | 2010 | 2011 | 2012 |
|-----|------|------|------|
| Gross Margin % | 30.4% | 21.9% | 11.4% |
| Operating Margin % | -1.1% | -20.4% | -29.1% |
| COGS as % of Revenue | 69.6% | 78.1% | 88.6% |
| OpEx as % of Revenue | 31.4% | 42.3% | 40.5% |
| **Total (COGS + OpEx) as % of Revenue** | **101.0%** | **120.4%** | **129.1%** |

### Shareholder Value

| Metric | 2010 | 2011 | 2012 |
|--------|------|------|------|
| EPS (Basic) | -$0.13 | -$2.00 | -$3.55 |
| EPS (Diluted) | -$0.13 | -$2.00 | -$3.55 |
| Shares Outstanding | 67,522K | 67,910K
 | 68,369K |


---

## 4. Data Vizualizations

<img width="591" height="470" alt="graph1" src="https://github.com/user-attachments/assets/6d2ad85b-6d5d-4736-978d-b0b629a42292" />
### Revenue and spending both dipped from 2010-2011, and then increased back up from 2011-2012. This sign of "less activity", whether it be laying off staff, producing less in-house, or simply publishing less, is likely a good indicator at predicting a game publishers dissolve.

<img width="989" height="590" alt="graph2" src="https://github.com/user-attachments/assets/7aeae3b6-a78c-4e17-8f2d-9ae31a5d63f8" />
### Royalty costs have consistantly been slightly under half of the companies total cost of goods, however in THQ's final year (2012) they had the pay more royalties for licneses that needed to be cancelled while their games were still actively being developed. This leads to an above 50% margin of royalty-to-total ratio that is a sign that a publisher is slowing down and is a useful predictor for a publishers dissolve.


---

**Research Date:** 2026-09-13  
**Data Period:** 2010-2012 Financial Statements (Fiscal years ending March 31)
