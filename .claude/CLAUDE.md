# Salesforce Claude reporting suite

## Purpose
Consistent OrderItem revenue retrieval, validation, and reporting for sales-report style analysis.

## Skill routing
For OrderItem / order product / revenue / sales-report / business-unit / product-family / account-hierarchy / sales-rep performance questions — including fiscal, YoY, and QoQ comparisons — use the `order-products` skill (`/order-products`). Complete its checklist before stating any totals.

Do **not** route Opportunity pipeline, open quotes, Cases, or non-revenue CRM tasks to that skill.

## Revenue defaults
- Inclusion: `Include_in_Sales_Reports__c = true` unless the user opts out
- Amount: `Sales_Report_Amount__c`
- Period: Income date/fiscal fields, not delivery date, unless the user asks about delivery
