# Salesforce Claude reporting suite

## Purpose
Consistent OrderItem revenue retrieval, validation, and reporting for sales-report style analysis against the production org (My Domain: `osg.my.salesforce.com`).

## Salesforce connection
- **MCP server:** Hosted `sobject-reads` only (read-only sObject access) — `salesforce-sobject-reads` in project `.mcp.json`
- **Endpoint:** `https://api.salesforce.com/platform/mcp/v1/platform/sobject-reads`
- **Auth:** External Client App OAuth (PKCE). Complete sign-in via `/mcp` in Claude Code if the server shows as needing authentication.
- **Policy:** Read-only. Do not attempt create, update, or delete. Prefer aggregate SOQL for large revenue questions.
- **Org:** Production. Treat all query results as sensitive business data.

If MCP is disconnected, say so and do not invent query results. Guide the user to `/mcp` to re-authenticate.

## Skill routing
For OrderItem / order product / revenue / sales-report / business-unit / product-family / account-hierarchy / sales-rep performance questions — including fiscal, YoY, and QoQ comparisons — use the `order-products` skill (`/order-products`). Complete its checklist before stating any totals.

Do **not** route Opportunity pipeline, open quotes, Cases, or non-revenue CRM tasks to that skill.

## Revenue defaults
- Inclusion: `Include_in_Sales_Reports__c = true` unless the user opts out
- Amount: `Sales_Report_Amount__c`
- Period: Income date/fiscal fields, not delivery date, unless the user asks about delivery
