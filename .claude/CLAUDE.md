# Salesforce Claude reporting suite

## Purpose
Consistent OrderItem revenue retrieval, validation, and reporting for sales-report style analysis against the production org (My Domain: `osg.my.salesforce.com`).

## Salesforce connection
- **MCP server:** Hosted `sobject-reads` only (read-only). Endpoint: `https://api.salesforce.com/platform/mcp/v1/platform/sobject-reads`
- **Org:** Production My Domain `osg.my.salesforce.com`. Treat results as sensitive.
- **Policy:** Read-only. Prefer aggregate SOQL for large revenue questions.
- **Important:** GitHub / Project file sync does **not** authenticate Salesforce. Live access requires a connected MCP session:
  - **Claude Code:** project `.mcp.json` server `salesforce-sobject-reads` + `/mcp` (callback `http://localhost:8080/callback` on the External Client App).
  - **Claude.ai Project:** Customize → Connectors → custom connector to the same server URL + OAuth Client ID/Secret (callback `https://claude.ai/api/mcp/auth_callback`). See `docs/SALESFORCE-MCP-SETUP.md`.
- If MCP is disconnected or tools are unavailable, say so clearly and do not invent query results.

## Skill routing
For OrderItem / order product / revenue / sales-report / business-unit / product-family / account-hierarchy / sales-rep performance questions — including fiscal, YoY, and QoQ comparisons — use the `order-products` skill (`/order-products`). Complete its checklist before stating any totals.

Do **not** route Opportunity pipeline, open quotes, Cases, or non-revenue CRM tasks to that skill.

## Revenue defaults
- Inclusion: `Include_in_Sales_Reports__c = true` unless the user opts out
- Amount: `Sales_Report_Amount__c`
- Period: Income date/fiscal fields, not delivery date, unless the user asks about delivery
