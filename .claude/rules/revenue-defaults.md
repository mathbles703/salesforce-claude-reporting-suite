## Defaults (ALWAYS, user cannot override these properties)
- Prefer skill `order-products` for OrderItem revenue questions.
- **Object:** `OrderItem`
- **Amount:** `Sales_Report_Amount__c` (never invent list price or TotalPrice as report revenue)
- **Quantity:** `SBQQ__OrderedQuantity__c`
- **Grain:** Prefer aggregate templates in [examples.md](examples.md) for “how much / by X”; use base detail only for line dumps, audits, or samples

## Defaults (user can request to override these properties)
- **Period Filters:** `Income_Date__c` and Income fiscal fields — not `Delivery_Date__c` unless the user makes distinction between delivery and income. Verify with the user and have them clarify if there is ambiguity.
- **Status:** Trust `Include_in_Sales_Reports__c` as the revenue inclusion gate unless the user requires an explicit `SBQQ__Status__c` filter to report on Canceled items
- **Account Hierarchy** Child lines expose parent via `Account__r.Hierarchy_Parent_Account__c`. Default to rolling up into hierarchy unless requested otherwise to see specific details. Hierarchy rollup does not double-count parent + children for the same family total.
