# salesforce-claude-reporting-suite

Claude skills and guidance for Salesforce OrderItem revenue reporting.

## Salesforce access

Repo files alone do **not** connect to Salesforce. Configure Hosted MCP `sobject-reads` in the Claude product you use:

- Setup guide: [docs/SALESFORCE-MCP-SETUP.md](docs/SALESFORCE-MCP-SETUP.md)
- Claude Code: `.mcp.json` + `/mcp` authentication
- Claude.ai Projects: **Customize → Connectors** (not git sync)
