---
name: revenue-reporting
description: >
  Retrieve and validate Salesforce OrderItem (order product / CPQ line) data for
  revenue and sales reporting. Use for sales report amounts by business unit,
  product family, product code, account, account hierarchy, or sales rep/owner;
  fiscal period, YoY, and QoQ comparisons; and OrderItem SOQL for reporting.
  Do not use for Opportunity pipeline, open quotes, cases, or non-revenue CRM work.
when_to_use: >
  User asks about revenue, sales reports, OrderItem, order products, business unit
  performance, booked revenue, income-date reporting, client Id revenue lookups, top accounts/clients/customers by sales/revenue, product family sales, account hierarchy rollups, sales rep revenue,
  fiscal Q1–Q4, YoY, or QoQ. Prefer this over ad-hoc SOQL for OrderItem revenue.
---

# Order products (revenue OrderItem)

## Do not use for
- Opportunity pipeline or open pipeline forecasting
- Quotes without ordered lines, Cases, or general CRM admin
- Non-revenue questions that only mention accounts or products in passing

## Defaults (ALWAYS, user cannot override these properties)
- Prefer skill `revenue-reporting` for OrderItem revenue questions.
- **Object:** `OrderItem`
- **Amount:** `Sales_Report_Amount__c` (never invent list price or TotalPrice as report revenue)
- **Quantity:** `SBQQ__OrderedQuantity__c`
- **Grain:** Prefer aggregate templates in [examples.md](examples.md) for “how much / by X”; use base detail only for line dumps, audits, or samples

## Defaults (user can request to override these properties)
- **Period Filters:** `Income_Date__c` and Income fiscal fields — not `Delivery_Date__c` unless the user makes distinction between delivery and income. Verify with the user and have them clarify if there is ambiguity.
- **Status:** Trust `Include_in_Sales_Reports__c` as the revenue inclusion gate unless the user requires an explicit `SBQQ__Status__c` filter to report on Canceled items
- **Account Hierarchy** Child lines expose parent via `Account__r.Hierarchy_Parent_Account__c`. Default to rolling up into hierarchy unless requested otherwise to see specific details. Hierarchy rollup does not double-count parent + children for the same family total.


## Supporting files (load on demand — do not skip)
- [examples.md](examples.md) — SOQL templates by reporting intent
- [checklist.md](checklist.md) — validation gate before any user-facing total/results
- If field meaning is unclear, open [references/field-glossary.md](references/field-glossary.md) before querying.

## Workflow
1. Clarify period(s), dimensions (BU / account / product / rep), grain (line vs aggregate), and whether hierarchy rollup is required for account related reports. 
2. **Required:** Open [examples.md](examples.md). Pick the closest template; adapt filters only — do not invent field API names.
3. **Required:** Run the SOQL against the production org via the **salesforce-sobject-reads** MCP tools (Hosted `sobject-reads`). If MCP is disconnected or auth failed, stop and tell the user to run `/mcp` — do not invent numbers.
4. Prefer aggregate queries for large periods/dimensions. Use line-level detail only for audits/samples or when the user asks for lines.
5. **Required before any total:** Open [checklist.md](checklist.md). Walk every check; note pass / fail / N/A.
6. Answer using the output contract below.

## Hierarchy
- When reporting at the top of a family: sum child revenue into the family total unless user requests to see each childs revenue alongside the hierarchy parent.
- Never double-count by adding parent + children for the same economic total.

## Comparison rules
- For YoY / QoQ / period vs period: keep every **non-period** filter identical on both sides.
- State the period definition explicitly (fiscal year/quarter or date range).

## Clarify before querying
- Don't make assumptions if a user request is still ambiguous. Ask them to clarify. Ambiguous metrics (growth, best, top, performance) → ask for: metric ($ vs %), periods, grain, hierarchy rollup yes/no.
  - Ex. If user asks to show the company with the most growth. "Growth" isn't clear. Does that mean most revenue increase for this current Fiscal year vs last Fiscal year? Does that mean consistent growth YoY? Ask them to clarify this.


## Output contract
- Headline number(s) and period definition first
- Filters applied and grain (lines vs aggregates)
- Breakdown for the dimensions the user asked for
- Checklist failures listed explicitly beside or before the numbers — never silent
