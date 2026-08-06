# salesforce-claude-reporting-suite
Create reliable and predictable structures to report on product revenue and the ability to determine quarterly goals, KPI metrics and repeatable reports/charting


## Salesforce access
Repo files alone do **not** connect to Salesforce. Configure Hosted MCP `sobject-reads` in the Claude product you use and ensure the project is connected using the custom connector:

- Claude.ai Projects: **Customize → Custom Connectors → Salesforce MCP**
    - On first run, Claude will ask user to authorize and force login. Must have a OSG Salesforce account
	- Permissions are based off user visibility. If they cannot view order products in Salesforce, Claude will not bypass that data policy