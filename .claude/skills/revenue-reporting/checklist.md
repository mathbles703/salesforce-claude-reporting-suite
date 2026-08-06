# Order products validation checklist

Complete **before** any user-facing revenue total. Mark each item **pass** / **fail** / **N/A**.

## Query construction
- Comparison queries share identical non-period filters on every side
- Field API names come from [examples.md](examples.md)


## Result integrity

- No duplicate `OrderItem` Ids in the working set
- Required dimensions present for the requested grain (`Product2.Business_Unit__c`, `Product2.ProductCode`, Account, period, owner, etc.)
- `SBQQ__OrderedQuantity__c > 0` (flag non-positive quantities)
- Null or zero `Sales_Report_Amount__c` lines flagged when they affect totals, counts, or averages
- Missing key fields for the ask are flagged (`ProductCode`, `Business_Unit__c`, `Income_Date__c`, `Account__c` as relevant) and records that are missing the relevant fields are considered incomplete. Note them in the result but do not include them.

## Response quality

- Period definition stated (fiscal year/quarter or date range)
- Grain stated (line detail vs aggregate)
- Filters stated (inclusion, BU, account, product, owner, etc.)
- Failures listed next to or before headline numbers
- Material outliers called out in an addendum (not silently mixed into result narrative)
- Outliers context given

## On failure

- Do not silently drop or “fix” numbers in prose.
- Prefer a corrected re-query over manual adjustment.
- If volume is too large for a safe line-level pull, switch to aggregate SOQL
