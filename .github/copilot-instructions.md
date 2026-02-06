# OctoCAT Supply Chain Management – Master Platform Instructions

This is the **master orchestrator** repository for the OctoCAT Supply Chain application. It does not contain application source code — it holds architecture docs, specifications, infrastructure configs, and Copilot Agent Skills that coordinate work across dependent repos.

## Ecosystem Repositories

| Repo | Purpose | Stack |
|------|---------|-------|
| `octocat-supply-platform` | Architecture, specs, infra, orchestration skills (this repo) | Markdown, Bicep, Docker |
| `octocat-supply-web` | Frontend UI | React 18 · Vite · Tailwind CSS · React Router v7 · React Query |
| `octocat-supply-api` | REST API | Express.js · SQLite · Repository pattern · Swagger/OpenAPI |

## Entity Model

Entities managed by the application:
- **Supplier** – provides products and deliveries
- **Product** – catalog items with SKU, price, stock level
- **Order** – placed at a Branch, contains OrderDetails
- **OrderDetail** – line item referencing a Product
- **Delivery** – from a Supplier, fulfilled via OrderDetailDelivery
- **OrderDetailDelivery** – junction linking OrderDetail ↔ Delivery
- **Headquarters** – parent organization
- **Branch** – belongs to a Headquarters, places Orders

## Agent Skills

This repo contains three orchestration skills in `.github/skills/`:

1. **`multi-repo-orchestration`** – Analyzes feature issues and spawns child issues in dependent repos.
2. **`cross-repo-pr-linking`** – Tracks PRs across repos and updates the master issue.
3. **`architecture-context`** – Provides entity model, API contracts, and component specs to agents in other repos.

## Workflow

1. Create a feature issue in this repo and assign to `@copilot`.
2. Copilot reads the dependency map and spawns issues in `octocat-supply-api` and/or `octocat-supply-web`.
3. Agents in each repo implement changes and report back via the `report-to-master` skill.
4. This repo's `cross-repo-pr-linking` skill maintains a live status table on the master issue.

## GitHub MCP Server Setup (Required for Cross-Repo Skills)

The orchestration skills in this repo require the GitHub MCP Server so Copilot coding agent can create issues, assign Copilot, and comment across repos.

### Setup Steps

| Step | Action |
|------|--------|
| 1 | Go to **Settings → Copilot → Coding agent → MCP configuration** and add the GitHub MCP server JSON config (see below) |
| 2 | Use `https://api.githubcopilot.com/mcp` as the server URL (not `/readonly`) for write access |
| 3 | Include `issues` in the `X-MCP-Toolsets` header so the agent can create and manage issues |
| 4 | Add a GitHub PAT as `COPILOT_MCP_GITHUB_PERSONAL_ACCESS_TOKEN` in the repo's **Copilot environment secrets** |
| 5 | Instruct the agent in your issue to create issues in target repos and assign Copilot |

### MCP Configuration JSON

```json
{
  "mcpServers": {
    "github": {
      "url": "https://api.githubcopilot.com/mcp",
      "headers": {
        "X-MCP-Toolsets": "issues"
      }
    }
  }
}
```

### PAT Scopes Required

The PAT stored in `COPILOT_MCP_GITHUB_PERSONAL_ACCESS_TOKEN` needs:
- `repo` — full control of private repositories (or `public_repo` for public-only)
- `issues:write` — create/update issues across repos

> **Note:** Without this setup, the `multi-repo-orchestration` and `cross-repo-pr-linking` skills will not be able to interact with other repositories.

## General Guidance

- Prefer incremental, minimal diffs; preserve existing style and naming.
- Surface security, correctness, and data integrity issues before micro-optimizations.
- Encourage type safety (no `any` unless justified).
- Flag duplicate logic that belongs in a shared utility or repository method.

## Escalation Order for Suggestions

1. Security / data integrity
2. Logical / functional correctness
3. Performance / scalability
4. Maintainability / duplication
5. Readability / consistency
6. Style / minor formatting
