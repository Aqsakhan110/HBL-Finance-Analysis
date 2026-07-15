# 🏦 HBL Finance Analysis — Power BI Dashboard

Real-time insights into **transactions, customers & risk**, built in Power BI from 50,000 transaction records and 5,000 customer profiles.

## 📌 Project Overview

This project analyzes HBL's financial transaction data to uncover patterns in transaction success/failure, customer segment behavior, regional performance, and fraud/risk signals. The goal was to turn raw transaction and customer CSVs into a clean, interactive dashboard that answers real business questions — where is money moving, where is it failing, and where is risk concentrated.

**Dataset:** `finance_transactions.csv` (50,000 rows, 15 columns) + `customers.csv` (5,000 rows, 10 columns)
**Tools used:** Power BI Desktop, Power Query (M), DAX
**Report type:** Transaction & Risk Performance Analysis

## 📊 Key KPIs (2026, Jan–Apr)

| KPI | Value |
|---|---|
| Total Transaction Amount | Rs 45.32M |
| Total Transactions | 4,993 |
| Average Transaction Value | Rs 9,077 |
| Total Fee Income | Rs 75.44K |
| Total Tax Collected | Rs 13.57K |
| Successful Transaction Rate | 85.88% |
| Failed Transaction Rate | 9.75% |
| Fraud-Flagged Rate | 1.28% |

## 🔍 Key Insights

- **Retail dominates** — the Retail segment contributes Rs 24.08M (53%) of total transaction value, more than every other segment combined.
- **Punjab leads regionally** — Rs 27.19M (60% of total value), followed by Sindh (Rs 8.88M); Balochistan and ICT are the smallest markets.
- **Loan EMI & Transfer are the biggest transaction types** — Rs 12.31M and Rs 12.28M respectively, over 54% of total value between them.
- **Loan EMI drives the most failures** — Rs 1.57M in failed value, the single largest contributor to the 9.75% failure rate.
- **Raast and ATM channels show the highest failed amounts**, hinting at possible connectivity/timeout issues on those rails.
- **Gender split is balanced** — Female customers (51.6%) vs Male customers (48.4%), unlike the highly concentrated segment/region splits.
- **Fraud is hard to catch on amount alone** — flagged transactions (Rs 8,827 avg) are close in size to normal ones (Rs 9,081 avg), suggesting amount-based rules aren't sufficient.

## 🖥️ Dashboard Pages

**1. Overview Analysis** — KPI cards with YoY comparison, monthly trend, transaction status donut, customer segment & state bar charts, transaction type matrix, and gender split.

**2. Transactions** — Row-level transaction detail table with full filtering (Year, Occupation, Merchant Category, Dynamic Metric).


## 🧹 Power Query — Data Cleaning

**`finance_transactions` table:**
- Fixed inconsistent date formatting (`-` → `/`) and parsed `transaction_date` into a proper Date type
- Corrected data types: `amount`, `fee_amount`, `tax_amount` → Number; `risk_score` → Whole Number
- Trimmed whitespace from the `channel` column (`Text.Trim`)
- Removed duplicate rows on `transaction_id` (`Table.Distinct`)
- Converted negative/inconsistent amounts to absolute values (`Number.Abs`)
- Replaced nulls in `fee_amount` with the column average (14.52) instead of dropping rows
- Reordered columns into a clean, logical schema

**`customers` table:**
- Fixed inconsistent formatting in `date_of_birth` and `join_date`
- Merged `fisrt_name` + `second_name` into a single `customr_name` column (`Table.CombineColumns`)
- Removed blank placeholder columns left over from the raw export
- Reordered and renamed columns for a presentation-ready schema

## 🧮 DAX Highlights

Every KPI card follows a repeatable **base → previous year → YoY → YoY%** pattern using `SAMEPERIODLASTYEAR` against a dedicated Calendar table:

```dax
Calender Table = CALENDAR(
    MIN(finance_transactions[transaction_Date]),
    MAX(finance_transactions[transaction_Date])
)

Total Amount measure M = SUM(finance_transactions[amount])

previous year amount =
CALCULATE([Total Amount measure M], SAMEPERIODLASTYEAR('Calender Table'[Date]))

YOY amount = [Total Amount measure M] - [previous year amount]

YOY % Amount = DIVIDE([YOY amount], [previous year amount])
```

The same 4-measure pattern is repeated for **Total Transactions**, **Average Transaction**, **Total Fee**, and **Total Tax** (16 measures total).

Formatted display measures for clean KPI card labels:

```dax
Total Amount = "Rs  " & FORMAT([Total Amount measure M]/1000000, "0.00") & "M"
Total Fee    = "Rs  " & FORMAT([Total Fee measure]/1000, "0.00") & "K"
```

A **Dynamic Metric** calculated table (using `NAMEOF()`) lets a single chart/card set switch between four measures from one slicer instead of building four separate visuals:

```dax
Dynamic Metric = {
    ("Total Amount ", NAMEOF('finance_transactions'[Total Amount measure M]), 0),
    ("Total Fee", NAMEOF('finance_transactions'[Total Fee measure]), 1),
    ("Total Tax", NAMEOF('finance_transactions'[Total Tax measure]), 2),
    ("Total Transactions", NAMEOF('finance_transactions'[Total Transactions]), 3)
}
```

## 💡 Recommendations

1. **Transaction reliability** — audit the Loan EMI auto-debit workflow (retry logic, mandate validity, timing vs. salary credit dates); investigate Raast/ATM channel failures.
2. **Customer strategy** — grow Corporate & Wealth segments through targeted cross-sell, since they contribute the least despite typically higher per-customer value.
3. **Regional growth** — increase branch/agent presence and marketing in Balochistan and ICT.
4. **Risk management** — move beyond amount-based fraud rules; add velocity, location, and channel-based signals.
5. **Reporting** — add a dedicated failed-transactions page filtered by type/channel for the operations team.

## 📁 Repository Structure

```
├── HBL_Finance_Analysis.pbix          # Power BI source file
├── HBL_Finance_Analysis_Report.docx   # Full analysis & recommendations report
├── README.md
── HBL Finance Dashboard.png
── HBL Finance Transaction Detail.png
  
```

## 👤 Author

**Aqsa Ayub**
Data Analyst | Power BI · Excel · Power Query · DAX
