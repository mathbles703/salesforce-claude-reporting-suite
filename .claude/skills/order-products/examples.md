# OrderItem revenue SOQL examples

Reference SOQL patterns for `OrderItem` revenue reporting.

## Conventions

- **Amount:** always aggregate or return `Sales_Report_Amount__c` as report revenue.
- **Inclusion:** every query includes `Include_in_Sales_Reports__c = true` unless the user opts out.
- **Quantity:** `SBQQ__OrderedQuantity__c`.
- **Period filters:** prefer `Income_Date__c` date literals (`THIS_FISCAL_YEAR`, `THIS_FISCAL_QUARTER`, `LAST_FISCAL_YEAR`, `N_FISCAL_QUARTERS_AGO:n`, etc.).
- **Period labels:** detail queries may select custom Income fiscal relationships (`Income_Fiscal_Quarter__r.*`, `Income_Fiscal_Month__r.*`) for display. Aggregate examples below use `FISCAL_YEAR` / `FISCAL_QUARTER` / `FISCAL_MONTH` on `Income_Date__c` for grouping. **Do not mix custom Income fiscal objects and standard fiscal functions in one comparison without reconciling.** If custom Income fiscal objects are the system of record for labels in this org, group by those relationship fields instead.
- **Delivery:** use `Delivery_Date__c` only when the user asks about delivery, not revenue recognition/income period.
- **Status:** `SBQQ__Status__c` is selected on detail for audit; do not add a status filter unless policy requires it — inclusion is gated by `Include_in_Sales_Reports__c`.
- **Comparisons:** keep every non-period filter identical across periods.
- **Volume:** prefer aggregates for “how much / by X”; use base detail only with a tight date window.

## Template index

| Need | Section |
|------|---------|
| Line detail / audit / export sample | Base detail query |
| By business unit | Revenue per business unit |
| By product family / product code | Revenue per product family and product code |
| By account / hierarchy | Revenue by account hierarchy |
| By sales rep / owner | Revenue per sales owner per month |
| FY vs last FY | Compare total revenue — last FY vs current FY |
| Same fiscal quarter last year | Compare total revenue — current FQ vs last year FQ |
| Scoped to one BU or account | Scoped filters (add-on) |

---

## Base detail query

Use when individual line items are required (audit, sample, export). Keep the date window tight.

```sql
SELECT
    Id,
    SBQQ__Status__c,
    Sales_Report_Amount__c,
    Account__r.Name,
    Account__r.Client_ID__c,
    Account__r.Hierarchy_Parent_Account__c,
    Product2.ProductCode,
    Product2.Family,
    Product2.Business_Unit__c,
    Delivery_Date__c,
    Order.Owner.Name,
    Income_Date__c,
    Income_Fiscal_Quarter__r.Fiscal_Year__c,
    Income_Fiscal_Month__r.Fiscal_Month__c,
    Income_Fiscal_Quarter__r.Fiscal_Quarter__c,
    SBQQ__OrderedQuantity__c
FROM OrderItem
WHERE Include_in_Sales_Reports__c = true
  AND Income_Date__c = LAST_N_FISCAL_QUARTERS:2
ORDER BY Income_Date__c DESC
```

---

## Revenue per business unit

Current fiscal year totals by `Product2.Business_Unit__c`.

```sql
SELECT
    Product2.Business_Unit__c BusinessUnit,
    SUM(Sales_Report_Amount__c) TotalRevenue,
    SUM(SBQQ__OrderedQuantity__c) TotalQty,
    COUNT(Id) LineCount
FROM OrderItem
WHERE Include_in_Sales_Reports__c = true
  AND Income_Date__c = THIS_FISCAL_YEAR
GROUP BY Product2.Business_Unit__c
ORDER BY SUM(Sales_Report_Amount__c) DESC
```

---

## Revenue per product family and product code

Current fiscal year by family and product.

```sql
SELECT
    Product2.Family ProductFamily,
    Product2.ProductCode ProductCode,
    SUM(Sales_Report_Amount__c) TotalRevenue,
    SUM(SBQQ__OrderedQuantity__c) TotalQty
FROM OrderItem
WHERE Include_in_Sales_Reports__c = true
  AND Income_Date__c = THIS_FISCAL_YEAR
GROUP BY Product2.Family, Product2.ProductCode
ORDER BY SUM(Sales_Report_Amount__c) DESC
```

---

## Revenue by account hierarchy

Current fiscal quarter by account and parent. For a **parent family total**, filter to the parent and its children, then **sum child lines** — do not add a pre-rolled parent row on top of children if that would double-count.

```sql
SELECT
    Account__r.Hierarchy_Parent_Account__c ParentAccountId,
    Account__c AccountId,
    Account__r.Name AccountName,
    Account__r.Client_ID__c ClientId,
    SUM(Sales_Report_Amount__c) TotalRevenue,
    COUNT(Id) LineCount
FROM OrderItem
WHERE Include_in_Sales_Reports__c = true
  AND Income_Date__c = THIS_FISCAL_QUARTER
GROUP BY
    Account__r.Hierarchy_Parent_Account__c,
    Account__c,
    Account__r.Name,
    Account__r.Client_ID__c
ORDER BY SUM(Sales_Report_Amount__c) DESC
```

Scoped family example (replace bind values):

```sql
SELECT
    Account__c AccountId,
    Account__r.Name AccountName,
    Account__r.Hierarchy_Parent_Account__c ParentAccountId,
    SUM(Sales_Report_Amount__c) TotalRevenue
FROM OrderItem
WHERE Include_in_Sales_Reports__c = true
  AND Income_Date__c = THIS_FISCAL_QUARTER
  AND (
      Account__c = '001XXXXXXXXXXXX'
      OR Account__r.Hierarchy_Parent_Account__c = '001XXXXXXXXXXXX'
  )
GROUP BY
    Account__c,
    Account__r.Name,
    Account__r.Hierarchy_Parent_Account__c
ORDER BY SUM(Sales_Report_Amount__c) DESC
```

---

## Revenue per sales owner per month

Current fiscal year by order owner and fiscal month.

```sql
SELECT
    Order.Owner.Name SalesOwner,
    FISCAL_MONTH(Income_Date__c) FiscalMonth,
    SUM(Sales_Report_Amount__c) TotalRevenue
FROM OrderItem
WHERE Include_in_Sales_Reports__c = true
  AND Income_Date__c = THIS_FISCAL_YEAR
GROUP BY Order.Owner.Name, FISCAL_MONTH(Income_Date__c)
ORDER BY Order.Owner.Name, FISCAL_MONTH(Income_Date__c)
```

---

## Compare total revenue — last FY vs current FY

```sql
SELECT
    FISCAL_YEAR(Income_Date__c) FiscalYear,
    SUM(Sales_Report_Amount__c) TotalRevenue
FROM OrderItem
WHERE Include_in_Sales_Reports__c = true
  AND (
      Income_Date__c = THIS_FISCAL_YEAR
      OR Income_Date__c = LAST_FISCAL_YEAR
  )
GROUP BY FISCAL_YEAR(Income_Date__c)
ORDER BY FISCAL_YEAR(Income_Date__c)
```

When adding BU, product, account, or owner filters, apply the **same** filters for both years.

---

## Compare total revenue — current FQ vs last year FQ

Same fiscal quarter one year ago (`N_FISCAL_QUARTERS_AGO:4`).

```sql
SELECT
    FISCAL_YEAR(Income_Date__c) FiscalYear,
    FISCAL_QUARTER(Income_Date__c) FiscalQuarter,
    SUM(Sales_Report_Amount__c) TotalRevenue
FROM OrderItem
WHERE Include_in_Sales_Reports__c = true
  AND (
      Income_Date__c = THIS_FISCAL_QUARTER
      OR Income_Date__c = N_FISCAL_QUARTERS_AGO:4
  )
GROUP BY FISCAL_YEAR(Income_Date__c), FISCAL_QUARTER(Income_Date__c)
ORDER BY FISCAL_YEAR(Income_Date__c), FISCAL_QUARTER(Income_Date__c)
```

---

## Scoped filters (add-on)

Add these to any template’s `WHERE` (and matching `GROUP BY` only if selecting the field). Keep them identical on both sides of a comparison.

```sql
-- Single business unit
AND Product2.Business_Unit__c = 'Hardware'

-- Product family
AND Product2.Family = 'Support'

-- Product code
AND Product2.ProductCode = 'SKU-123'

-- Account
AND Account__c = '001XXXXXXXXXXXX'

-- Sales owner (name or Id — prefer Id when known)
AND Order.Owner.Name = 'Jane Rep'
```

Optional amount hygiene (only if policy excludes zero/null report amounts from totals):

```sql
AND Sales_Report_Amount__c != null
AND Sales_Report_Amount__c > 0
```
