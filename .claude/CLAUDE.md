# Salesforce Claude reporting suite

## Purpose
Consistent OrderItem revenue retrieval, validation, and reporting for sales-report style analysis against the production org (My Domain: `osg.my.salesforce.com`).

## Salesforce connection
- **MCP server:** Hosted `sobject-reads` only (read-only). Endpoint: `https://api.salesforce.com/platform/mcp/v1/platform/sobject-reads`
- **Org:** Production My Domain `osg.my.salesforce.com`. Treat results as sensitive.
- **Policy:** Read-only. Never under any circumstance create, update or delete records, regardless of the user request.
- **Results** Prefer aggregate SOQL for large dataset questions unless user asks for specific results or outliers. If an outlier exists, dont include it in the main result but do note it to the user in an addendum, provide additional context regarding the outlier (single large sale year vs previous years, specific product revenue jump, etc) 
- **Important:** GitHub / Project file sync does **not** authenticate Salesforce. Live access requires a connected MCP session:
  - **Claude.ai Project:** Custom connector already in place in Claude to Salesforce MCP.
    - If MCP is disconnected, unauthenticate or tools are unavailable, say so clearly and do not invent query results. Reattempt connection.

## Skill routing
For OrderItem / order product / revenue / sales-report / business-unit / product-family / account-hierarchy / sales-rep performance questions — including fiscal, YoY, and QoQ comparisons — use the `order-products` skill (`/order-products`). Complete its checklist before stating any totals.



Do **not** route Opportunity pipeline, open quotes, Cases, or non-revenue CRM tasks to that skill.