# Salesforce Hosted MCP setup (sobject-reads)

This project’s skills and SOQL templates live in git. **Salesforce connectivity does not.**  
Pushing to GitHub and attaching files to a Claude Project does **not** log you into Salesforce.

You must connect Hosted MCP **in the Claude product you actually use**.

---

## Which Claude product are you using?

| Product | How MCP is configured | Uses project `.mcp.json`? |
|---------|------------------------|---------------------------|
| **Claude Code** (terminal / IDE CLI) | Project `.mcp.json` + `/mcp` auth | **Yes** |
| **Claude.ai** (web Project / chat) | **Customize → Connectors** (custom connector) | **No** |
| **Claude Desktop** | Connectors / MCP settings in the app | **No** (unless you also run Code) |

If you “synced the repo to a Claude Project” on **claude.ai**, follow **Path A**.  
If you use **Claude Code** in this repo, follow **Path B**.

---

## Prerequisites (Salesforce — both paths)

Confirm all of these in the **production** org (`osg.my.salesforce.com`):

1. **MCP Service** enabled (Setup → User Interface / MCP settings).
2. Hosted server **sobject-reads** activated.
3. **External Client App** (not a classic Connected App only) with:
   - OAuth enabled
   - Scopes: **Access MCP servers** + **Perform requests at any time** (refresh token)
   - **PKCE** required/enabled
   - Correct **callback URL(s)** for the client you use (see below)
   - Your user **allowed** (if “Admin approved users”, pre-authorize yourself)
4. Your user can **read** `OrderItem` and related fields used by the skill.
5. After creating/changing the External Client App, wait up to **~30 minutes** if auth fails immediately.

### Callback URLs (add every client you use)

| Client | Callback URL to register on the External Client App |
|--------|------------------------------------------------------|
| Claude.ai / Claude Desktop (custom connector) | `https://claude.ai/api/mcp/auth_callback` |
| Claude Code (this repo’s `.mcp.json`, port 8080) | `http://localhost:8080/callback` |

You can list **both** on the same app (one per line).

### Server URL (production sobject-reads)

Primary (current GA pattern used in this project):

```text
https://api.salesforce.com/platform/mcp/v1/platform/sobject-reads
```

If Claude reports 404 / unknown server, try the alternate production form some docs list:

```text
https://api.salesforce.com/platform/mcp/v1/sobject-reads
```

Do **not** use sandbox URLs (`/v1/sandbox/...`) against production.

---

## Path A — Claude.ai Project (web)

Repo files (skills, CLAUDE.md, examples) load as **project knowledge**.  
They do **not** open an org session. Connect Salesforce separately:

### 1. Add a custom connector

1. Open [claude.ai](https://claude.ai) → **Customize** (or Settings).
2. **Connectors** → **+** → **Add custom connector**.
3. Name: e.g. `Salesforce sobject-reads (OSG prod)`.
4. **Server URL:**  
   `https://api.salesforce.com/platform/mcp/v1/platform/sobject-reads`
5. Open **Advanced** / OAuth settings if shown:
   - **OAuth Client ID** = External Client App **Consumer Key**
   - **OAuth Client Secret** = **Consumer Secret** (required by many Salesforce setups)
6. Save.

### 2. Connect / authenticate

1. On the connector, click **Connect** / **Authenticate**.
2. Complete Salesforce login for **production** (My Domain: `osg.my.salesforce.com`).
3. Approve access for the External Client App.
4. Confirm the connector shows **Connected**.

### 3. Use it in the Project

1. Open your Claude **Project** that has this repo’s files.
2. Ensure the connector is **enabled** for the project/chat (connector toggles vary by plan).
3. Ask:  
   `Using the Salesforce connector, list tools available`  
   or  
   `Run SOQL: SELECT Id FROM OrderItem WHERE Include_in_Sales_Reports__c = true LIMIT 3`

### Common Claude.ai failures

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| No Salesforce tools at all | Connector never added; only repo synced | Path A steps 1–3 |
| Redirect / invalid_client | Wrong Client ID or app not External Client App | Recheck Consumer Key; use External Client App |
| redirect_uri mismatch | Callback is only localhost | Add `https://claude.ai/api/mcp/auth_callback` |
| invalid_scope | Missing MCP scopes | Add **Access MCP servers** + refresh token scope |
| 401 after login | User not permitted on app / MCP not enabled | Pre-authorize user; enable MCP Service |
| Connected but SOQL fails | FLS / field-level security | Grant read on OrderItem + custom fields |
| “I can’t access Salesforce” in Project | Connector off for that chat/project | Enable connector; re-auth |

**Important:** Client Secret is often required on Claude.ai even when Claude Code can use public client + PKCE. Keep the secret out of git; paste only into Claude’s connector UI.

---

## Path B — Claude Code (this repository)

### 1. Project config

`.mcp.json` already defines:

- Name: `salesforce-sobject-reads`
- URL: production `sobject-reads`
- OAuth Client ID + `callbackPort: 8080`

### 2. External Client App callback

Must include:

```text
http://localhost:8080/callback
```

(`https://claude.ai/api/mcp/auth_callback` alone is **not** enough for Claude Code.)

### 3. Trust + login

```bash
cd /path/to/salesforce-claude-reporting-suite
claude
```

- Accept workspace trust if prompted.
- Approve project MCP from `.mcp.json` if prompted.
- Run `/mcp` → authenticate **salesforce-sobject-reads**.
- Or: `claude mcp login salesforce-sobject-reads`

If Salesforce requires a client secret:

```bash
MCP_CLIENT_SECRET='your-consumer-secret' claude mcp login salesforce-sobject-reads
```

Secret is stored in the OS keychain / Claude credentials store — **not** in git.

### 4. Verify

```text
/mcp
```

Server should show **connected** with tools. Then:

```text
Query OrderItem: SELECT Id, Sales_Report_Amount__c FROM OrderItem
WHERE Include_in_Sales_Reports__c = true LIMIT 5
```

---

## What git / Project sync *does* vs *does not*

| Synced from GitHub | Effect |
|--------------------|--------|
| Skills, checklist, examples, CLAUDE.md | How Claude **should** query and validate |
| `.mcp.json` | Used by **Claude Code** only after local trust + OAuth |
| Client Secret | Must **never** be in git |
| Live org session | **Never** created by git sync alone |

---

## Minimal checklist if “still unable to connect”

1. Which UI fails: **claude.ai Project** or **Claude Code**?
2. Exact error text (redirect_uri, invalid_client, 401, 404, invalid_scope, …).
3. External Client App has the **correct callback** for that product.
4. Consumer Key matches; Consumer Secret entered where the product asks for it.
5. Scopes include **Access MCP servers**.
6. Your user is allowed to use the app.
7. Server URL is production `sobject-reads` (not sandbox, not sobject-all if you only activated reads).
8. You completed **Connect/Authenticate** after adding the connector (not only saved the URL).

---

## After it connects

Use the `order-products` skill. It will:

1. Pick SOQL from `examples.md`
2. Run it via **sobject-reads** MCP tools
3. Validate with `checklist.md` before totals
