# Order products validation checklist

Complete **before** any user-facing revenue total. Mark each item **pass** / **fail** / **N/A**.

## Query construction

1. [ ] `Include_in_Sales_Reports__c = true` unless the user explicitly opted out
2. [ ] Amount field is `Sales_Report_Amount__c` (not list price, UnitPrice, or TotalPrice as report revenue)
3. [ ] Period uses `Income_Date__c` / Income fiscal fields — not `Delivery_Date__c` — unless the user asked about delivery
4. [ ] Comparison queries share identical non-period filters on every side
5. [ ] Field API names come from [examples.md](examples.md) or confirmed schema — none invented
6. [ ] Status handling matches policy (default: inclusion flag only; extra `SBQQ__Status__c` filter only if required)

## Result integrity

7. [ ] No duplicate `OrderItem` Ids in the working set
8. [ ] Required dimensions present for the requested grain (`Product2.Business_Unit__c`, `Product2.ProductCode`, Account, period, owner, etc.)
9. [ ] `SBQQ__OrderedQuantity__c > 0` (flag non-positive quantities)
10. [ ] Null or zero `Sales_Report_Amount__c` lines flagged when they affect totals, counts, or averages
11. [ ] Hierarchy rollup does not double-count parent + children for the same family total
12. [ ] Missing key fields for the ask are flagged (`ProductCode`, `Business_Unit__c`, `Income_Date__c`, Account as relevant)

## Response quality

13. [ ] Period definition stated (fiscal year/quarter or date range)
14. [ ] Grain stated (line detail vs aggregate)
15. [ ] Filters stated (inclusion, BU, account, product, owner, etc.)
16. [ ] Failures listed next to or before headline numbers

## On failure

- Do not silently drop or “fix” numbers in prose.
- Prefer a corrected re-query over manual adjustment.
- If volume is too large for a safe line-level pull, switch to aggregate SOQL or the org PHP batch path and say so.
