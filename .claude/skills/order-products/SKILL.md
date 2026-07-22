---
name: order-products
description: >
  Retrieve and validate Salesforce OrderItem (order product / CPQ line) data for
  revenue and sales reporting. Use for sales report amounts by business unit,
  product family, product code, account, account hierarchy, or sales rep/owner;
  fiscal period, YoY, and QoQ comparisons; and OrderItem SOQL for reporting.
  Do not use for Opportunity pipeline, open quotes, cases, or non-revenue CRM work.
when_to_use: >
  User asks about revenue, sales reports, OrderItem, order products, business unit
  performance, product family sales, account hierarchy rollups, sales rep revenue,
  fiscal Q1–Q4, YoY, or QoQ. Prefer this over ad-hoc SOQL for OrderItem revenue.
---

# Order products (revenue OrderItem)

## Do not use for
- Opportunity pipeline or open pipeline forecasting
- Quotes without ordered lines, Cases, or general CRM admin
- Non-revenue questions that only mention accounts or products in passing

## Defaults (unless user overrides)
- **Object:** `OrderItem`
- **Inclusion:** `Include_in_Sales_Reports__c = true`
- **Amount:** `Sales_Report_Amount__c` (never invent list price or TotalPrice as report revenue)
- **Period:** `Income_Date__c` and Income fiscal fields — not `Delivery_Date__c` unless the user asks about delivery
- **Quantity:** `SBQQ__OrderedQuantity__c`
- **Grain:** Prefer aggregate templates in [examples.md](examples.md) for “how much / by X”; use base detail only for line dumps, audits, or samples
- **Status:** Trust `Include_in_Sales_Reports__c` as the revenue inclusion gate unless the user or org policy requires an explicit `SBQQ__Status__c` filter

## Supporting files (load on demand — do not skip)
- [examples.md](examples.md) — SOQL templates by reporting intent
- [checklist.md](checklist.md) — validation gate before any user-facing total

## Workflow
1. Clarify period(s), dimensions (BU / account / product / rep), grain (line vs aggregate), and whether hierarchy rollup is required.
2. **Required:** Open [examples.md](examples.md). Pick the closest template; adapt filters only — do not invent field API names.
3. **Required:** Run the SOQL against the production org via the **salesforce-sobject-reads** MCP tools (Hosted `sobject-reads`). If MCP is disconnected or auth failed, stop and tell the user to run `/mcp` — do not invent numbers.
4. Prefer aggregate queries for large periods/dimensions. Use line-level detail only for audits/samples or when the user asks for lines.
5. **Required before any total:** Open [checklist.md](checklist.md). Walk every check; note pass / fail / N/A.
6. Answer using the output contract below.

## Hierarchy
- Child lines expose parent via `Account__r.Hierarchy_Parent_Account__c`.
- When reporting at the top of a family: sum child revenue into the family total.
- Never double-count by adding parent + children for the same economic total.

## Comparison rules
- For YoY / QoQ / period vs period: keep every **non-period** filter identical on both sides.
- State the period definition explicitly (fiscal year/quarter or date range).

## Output contract
- Headline number(s) and period definition first
- Filters applied and grain (lines vs aggregates)
- Breakdown for the dimensions the user asked for
- Checklist failures listed explicitly beside or before the numbers — never silent
