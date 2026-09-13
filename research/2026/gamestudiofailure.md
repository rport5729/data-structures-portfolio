# How Can Data Monitor The Stages of a Game Publisher's Failure?

---

## 1. Problem Definition

### Research Question
How can financial data from consolidated income statements be used to identify and monitor the distinct stages of a game publisher's decline and failure?

### Context
This case study examines **THQ** (creators of Saints Row and Darksiders franchises), a major game publisher that declined rapidly between 2010-2012. Understanding the financial indicators that signal organizational failure is critical for:
- Investors evaluating business health
- Company leadership making strategic decisions
- Stakeholders anticipating financial crises
- Researchers studying business failure patterns

### Relevance
Game publishers face unique pressures: volatile revenue from hit-driven products, high amortization costs from intellectual property portfolios, and significant operating expenses (R&D, marketing). Identifying failure patterns in this industry can help predict corporate bankruptcy, guide investment decisions, and inform turnaround strategies.

### Specific Research Questions
1. What financial patterns emerge in the years leading to corporate failure?
2. Can we identify distinct "failure stages" through income statement analysis?
3. Which KPIs best predict critical failure (Stage 3+)?
4. How effective are recovery attempts (restructuring, increased sales spending) when companies are already in decline?

---

## 2. Data Description

### Key Variables and Operationalization

| Variable | Definition | Measurement | Unit |
|----------|-----------|-------------|------|
| **Revenue** | Total sales from product sales | Annual reported revenue | $K (thousands) |
| **Product Costs** | Direct manufacturing/distribution costs | COGS line item | $K |
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

```python
# Gross Profit calculation
df['Gross_Profit'] = df['Revenue'] - df['Total_COGS']

# Margin percentages
df['Gross_Margin_Pct'] = (df['Gross_Profit'] / df['Revenue']) * 100
df['Operating_Margin_Pct'] = (df['Operating_Income'] / df['Revenue']) * 100

# Cost ratios
df['COGS_as_Pct_Revenue'] = (df['Total_COGS'] / df['Revenue']) * 100
df['OpEx_as_Pct_Revenue'] = (df['Total_OpEx'] / df['Revenue']) * 100

# EPS ratios
df['EPS_Change_YoY'] = df['EPS_Basic'].pct_change() * 100
```

**Rationale:** Percentage-based metrics normalize across years and allow for trend identification independent of absolute revenue changes.

### Step 3: Derived Failure Stage Indicators
**Variables Created:**

```python
# Operating expense ratio
df['OpEx_Revenue_Ratio'] = df['Total_OpEx'] / df['Revenue']

# Restructuring intensity
df['Restructuring_as_Pct_Revenue'] = (df['Restructuring_Costs'] / df['Revenue']) * 100

# Cost of sales intensity
df['COGS_Revenue_Ratio'] = df['Total_COGS'] / df['Revenue']

# Profitability trajectory
df['Gross_Margin_Decline'] = df['Gross_Margin_Pct'].diff()
df['Operating_Loss_Acceleration'] = df['Operating_Income'].diff()
```

**Rationale:** These composite indicators flag warning signs: when COGS + OpEx exceed 100% of revenue, a company is spending more than it earns and cannot be sustained.

### Step 4: Data Validation
**Checks Performed:**

```python
# Verify accounting identity
assert (df['Revenue'] - df['Total_COGS'] == df['Gross_Profit']).all()
assert (df['Gross_Profit'] - df['Total_OpEx'] == df['Operating_Income']).all()

# Verify no null values in critical fields
assert df[['Revenue', 'Total_COGS', 'Operating_Income']].isnull().sum() == 0

# Verify logical consistency
assert (df['Total_COGS'] > 0).all()  # COGS should be positive
assert (df['Revenue'] > 0).all()  # Revenue should be positive
```

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
| Shares Outstanding | 67,522K | 67,910K | 68,369K |

---

**Research Date:** 2026-09-13  
**Data Period:** 2010-2012 Financial Statements (Fiscal years ending March 31)
