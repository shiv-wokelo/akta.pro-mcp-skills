# akta.pro Skills for AI Agents

This repository hosts skill files that help AI agents use akta.pro’s private company data and news signals effectively once connected via MCP. A skill file teaches an agent what data is available and when to reach for it. The MCP connector is what actually gives the agent access to that data. akta.pro’s MCP server works with any MCP-compatible client, including Claude, ChatGPT, Claude Code, Cursor, VS Code, and OpenCode.

## What is akta.pro

akta.pro is a private company data and signals API built for the agent economy. It covers 20M+ private companies with 70+ structured data points each, alongside a news and signals layer that tracks companies, industries, and custom topics in natural language. Everything is entity-resolved to the same company identifiers and priced pay-as-you-go, with access through API, MCP, CLI, and bulk export.

## Capabilities available through these skills

### Company Database

Structured, queryable data across 70+ fields and 15 categories, including:
- Firmographics: legal name, website, founding year, operating status, headcount range
- Business model: GTM motion, revenue model, pricing details, distribution channels
- Financial estimates: revenue and valuation ranges
- Funding detail: funding rounds, investors, funding overview
- Company assessment: market position, strengths, weaknesses, competitive moat, key risks
- Management profiles: executive background and commentary
- Technology: tech stack, integrations, AI capabilities, app presence
- M&A and investment activity
- Company hierarchy: subsidiaries and ownership relationships

### Alternative Signals

Entity-resolved data between the lines, including headcount trends, web traffic, job posts, social posts, employee reviews, and product reviews.

### News and Signals

Three feed types under one schema, all deduplicated, entity-resolved, and scored for impact and sentiment:
- **Company News**: monitor any company across 20M+ entities, resolved to a single ID across every publisher
- **Industry News**: track any sector across 30,000+ sub-sectors described in plain language
- **Custom Signals**: turn any topic or market theme into a structured feed using natural language

Every record includes an AI summary, sentiment classification, impact scoring, event categorization, and full source provenance.

## Getting started

### 1. Create an akta.pro account

If you don’t already have one, go to playground.akta.pro/signup and create an account. New accounts come with 25 free credits, enough to try company profiles and news signals before committing to a plan.

### 2. Connect your agent

From your akta.pro dashboard, click **Connect Agent**. This opens the “Connect via MCP” panel, where you’ll find the server URL and setup steps for each supported client. Any MCP-compatible agent can connect using the same server URL below.

**Server URL (same for every client):**

```
https://mcp.akta.pro/mcp
```

Choose your client below and follow the matching steps.

---

#### Claude

1. Open Claude and go to Settings from the sidebar.
2. Select Connectors, then Add custom connector.
3. Name it “akta-pro” and paste the server URL above.
4. Click Add and complete authentication when prompted.

#### ChatGPT

1. Go to Settings → Apps, then enable Developer Mode.
2. Open Apps and click Create to start a new custom app.
3. Name it “akta-pro” and paste the server URL above, with OAuth as the authentication.
4. Click Create and complete authentication when prompted.

#### Claude Code

1. Open your terminal.
2. Run:
    
    ```
    claude mcp add --transport http akta-pro https://mcp.akta.pro/mcp --header "x-api-key: <YOUR_API_KEY>"
    ```
    
3. Replace `<YOUR_API_KEY>` with your akta.pro API key from the API Keys page.
4. Claude Code will confirm the server was added. No restart needed.

#### Cursor

1. Open or create the file `~/.cursor/mcp.json`.
2. Add an entry under `mcpServers` with the key `akta-pro`, the server URL above, and your API key in the `x-api-key` header.
3. Save the file.
4. Restart Cursor to activate the connector.

#### VS Code

1. Open or create `.vscode/mcp.json` in your project root.
2. Add an entry under `servers` with the key `akta-pro`, type `http`, the server URL above, and your API key in the `x-api-key` header.
3. Save the file.
4. Reload the VS Code window to activate.

#### OpenCode

1. Open or create `opencode.json` in your project root.
2. Add an entry under `mcp` with the key `akta-pro`, type `remote`, the server URL above, and `"enabled": true`.
3. Save the file.
4. Restart OpenCode to activate the connector.

---

### 3. Add the skill file

Place the relevant `SKILL.md` file from this repository wherever your agent looks for skills, or reference it according to your setup. Once both the connector and skill file are in place, ask your agent a company research question, and the skill will trigger automatically.

## Repository structure

```
README.md
/skills
  akta.pro MCP - competitive intelligence - skill.md
/samples
  akta.pro MCP - competitive intelligence - sample.html
```

## Learn more

- Website: https://akta.pro
- API Docs: https://docs.akta.pro/
- Playground: https://playground.akta.pro/
- Pricing: https://akta.pro/pricing
