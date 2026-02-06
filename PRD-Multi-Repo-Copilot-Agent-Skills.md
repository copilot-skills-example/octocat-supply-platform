# Product Requirements Document (PRD)

## Multi-Repository Copilot Agent Skills Orchestration

| Field | Value |
|-------|-------|
| **Document Version** | 2.0 |
| **Date** | February 6, 2026 |
| **Status** | Draft |
| **Author** | Platform Team |

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [Problem Statement](#2-problem-statement)
3. [Goals & Objectives](#3-goals--objectives)
4. [Solution Overview](#4-solution-overview)
5. [System Architecture](#5-system-architecture)
6. [Technical Specifications](#6-technical-specifications)
7. [Repository Structure](#7-repository-structure)
8. [Agent Skills Definitions](#8-agent-skills-definitions)
9. [Workflow Sequence](#9-workflow-sequence)
10. [Implementation Guide](#10-implementation-guide)
11. [Success Metrics](#11-success-metrics)
12. [Dependencies & Prerequisites](#12-dependencies--prerequisites)
13. [Risks & Mitigations](#13-risks--mitigations)

---

## 1. Executive Summary

This PRD outlines an architecture for orchestrating GitHub Copilot coding agents across multiple dependent repositories using **Agent Skills** with **MCP (Model Context Protocol)** servers for the **OctoCAT Supply Chain Management** application. When a feature issue is created in the master orchestrator repository (`octocat-supply-platform`) and assigned to Copilot, the system automatically spawns related issues in dependent repositories — the React frontend (`octocat-supply-web`), the Express.js API (`octocat-supply-api`), and optionally shared types (`octocat-supply-common`) — assigns Copilot agents to each, and maintains traceability by linking all resulting PRs back to the original master issue.

### Key Value Proposition

- **Automated cross-repo coordination** for multi-service supply chain features
- **Intelligent task decomposition** based on architecture dependencies (frontend → API → database)
- **Full traceability** from master issue → spawned issues → PRs
- **Zero manual orchestration** required for routine multi-repo changes
- **Domain-aware routing** — skills understand the OctoCAT entity model (Suppliers, Products, Orders, Deliveries, Branches, Headquarters)

---

## 2. Problem Statement

### Current Challenges

Modern software architectures often span multiple repositories. The OctoCAT Supply Chain application is no exception:
- **Frontend** — React 18+ / Vite / Tailwind CSS single-page application
- **API** — Express.js REST API with SQLite persistence and repository pattern
- **Shared types/contracts** — TypeScript interfaces and DTOs used by both layers
- **Infrastructure** — Bicep templates, Docker Compose, and GitHub Actions CI/CD

When implementing features that touch multiple repositories:

1. **Manual coordination** is required to create issues in each repo
2. **Context is lost** between repositories
3. **Tracking progress** across repos is difficult
4. **Integration order** must be manually determined
5. **PR linking** requires manual effort

### Impact

- Increased lead time for cross-cutting features
- Risk of misaligned implementations across repos
- Difficulty tracking feature completion status
- Developer context-switching overhead

---

## 3. Goals & Objectives

### Primary Goals

| Goal | Description | Success Criteria |
|------|-------------|------------------|
| **G1** | Automate multi-repo task spawning | Single issue creates linked issues in all affected repos |
| **G2** | Maintain full traceability | All PRs automatically linked to master issue |
| **G3** | Leverage Copilot agent capabilities | Agents handle implementation in each repo |
| **G4** | Zero-configuration for developers | Skills work out-of-the-box with standard setup |

### Non-Goals

- Replacing human code review
- Automatic merging of PRs
- Cross-repo transaction management
- Real-time synchronization between repos

---

## 4. Solution Overview

### Approach

Implement **GitHub Agent Skills** (following the [open standard](https://github.com/agentskills/agentskills)) that enable Copilot coding agents to:

1. **Analyze** feature requests against a dependency map
2. **Spawn** task issues in dependent repositories
3. **Track** progress across all related PRs
4. **Report** status back to the master issue

### Technology Stack

| Component | Technology |
|-----------|------------|
| Agent Runtime | GitHub Copilot Coding Agent |
| Skill Format | Agent Skills (`.github/skills/`) |
| External Tools | GitHub MCP Server |
| Configuration | YAML frontmatter + Markdown |
| Frontend Stack | React 18+ · Vite · Tailwind CSS · React Router v7 · React Query |
| API Stack | Express.js · SQLite · Swagger/OpenAPI · Repository pattern |
| Infrastructure | Docker Compose · Bicep (Azure Container Apps) · GitHub Actions |

---

## 5. System Architecture

### High-Level Architecture Diagram

```
┌──────────────────────────────────────────────────────────────────────┐
│              MASTER REPO (octocat-supply-platform)                   │
│  ┌───────────────────────────────────────────────────────────────┐   │
│  │  Issue: "Add supplier status dashboard"                       │   │
│  │  Assignee: @copilot                                           │   │
│  └───────────────────────────────────────────────────────────────┘   │
│                                │                                     │
│  .github/skills/                                                     │
│  ├── multi-repo-orchestration/             # Spawn issues            │
│  │   ├── SKILL.md                                                    │
│  │   └── dependencies.json                                          │
│  ├── cross-repo-pr-linking/                # Link PRs                │
│  │   └── SKILL.md                                                    │
│  └── architecture-context/                 # Fetch specs             │
│      └── SKILL.md                                                    │
│                                                                      │
│  architecture/                                                       │
│  ├── README.md              # System overview & ERD                  │
│  ├── entity-model.md        # Suppliers, Products, Orders, etc.      │
│  └── components/            # Component specs                        │
│  specs/                                                              │
│  ├── api/                   # OpenAPI / Swagger contracts            │
│  └── features/              # Feature requirement docs               │
└──────────────────────────────────────────────────────────────────────┘
                                │
                      Uses GitHub MCP Server tools:
                      • create_issue
                      • add_issue_comment
                      • get_file_contents
                      • search_issues
                                │
          ┌─────────────────────┼──────────────────────┐
          ▼                     ▼                      ▼
┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐
│  FRONTEND REPO   │  │  API REPO        │  │  SHARED REPO     │
│ octocat-supply-  │  │ octocat-supply-  │  │ octocat-supply-  │
│ web              │  │ api              │  │ common           │
│                  │  │                  │  │                  │
│ React 18 + Vite  │  │ Express.js +     │  │ Shared TS types  │
│ Tailwind CSS     │  │ SQLite           │  │ DTOs & contracts │
│ React Router v7  │  │ Repository       │  │ Utility fns      │
│ React Query      │  │ pattern          │  │                  │
│                  │  │ Swagger/OpenAPI  │  │                  │
│ .github/skills/  │  │ .github/skills/  │  │ .github/skills/  │
│ ├── report-to-   │  │ ├── report-to-   │  │ └── report-to-   │
│ │   master/      │  │ │   master/      │  │     master/      │
│ └── api-endpoint/│  │ └── api-endpoint/│  │                  │
└──────────────────┘  └──────────────────┘  └──────────────────┘
```

### OctoCAT Entity Model (shared across repos)

```
  Headquarters ──┐
                 ├──▶ Branch ──▶ Order ──▶ OrderDetail ──▶ OrderDetailDelivery
  Supplier ──────┤                              │                  │
                 └──▶ Delivery ─────────────────────────────────────┘
                                          Product ◄────────┘
```

### Data Flow

```
┌──────────┐     ┌───────────────────┐     ┌──────────────┐     ┌────────────────────┐
│  Human   │────▶│ Master Issue in   │────▶│   Copilot    │────▶│ Spawned Issues      │
│ creates  │     │ octocat-supply-   │     │ loads multi- │     │ • octocat-supply-  │
│ issue    │     │ platform          │     │ repo-orch    │     │   common (types)   │
│          │     │ @copilot assigned │     │ skill        │     │ • octocat-supply-  │
└──────────┘     └───────────────────┘     └──────────────┘     │   api (endpoints)  │
                                                                 │ • octocat-supply-  │
                                                                 │   web (UI)         │
                                                                 └────────────────────┘
                                                                          │
                                                                          ▼
┌──────────┐     ┌───────────────────┐     ┌──────────────┐     ┌────────────────────┐
│  Human   │◀────│ Master Issue      │◀────│   Copilot    │◀────│   PRs created       │
│ reviews  │     │ updated with      │     │ reports back │     │   in each repo      │
│ PRs      │     │ PR tracking table │     │              │     │                    │
└──────────┘     └───────────────────┘     └──────────────┘     └────────────────────┘
```

---

## 6. Technical Specifications

### Agent Skills Format

Based on the [official GitHub documentation](https://docs.github.com/en/enterprise-cloud@latest/copilot/concepts/agents/about-agent-skills):

| Element | Requirement |
|---------|-------------|
| Location | `.github/skills/<skill-name>/` or `.claude/skills/<skill-name>/` |
| Main File | `SKILL.md` (required name) |
| Frontmatter | YAML with `name` (required), `description` (required), `license` (optional) |
| Body | Markdown instructions for Copilot |
| Resources | Additional files in skill directory (scripts, configs, examples) |

### MCP Server Tools Used

| Tool | Purpose | Repository |
|------|---------|------------|
| `create_issue` | Spawn issues in dependent repos | Master |
| `add_issue_comment` | Update tracking on master issue | All |
| `get_file_contents` | Fetch architecture context | All |
| `search_issues` | Find related PRs | Master |
| `create_pull_request` | Create PRs with proper linking | Dependent |
| `list_workflow_runs` | Check CI status | All |

### Skill Trigger Mechanism

Copilot automatically loads skills based on:
1. **Description matching** - Skill description matches user prompt/issue content
2. **Explicit invocation** - User mentions skill by name
3. **Context relevance** - Copilot determines skill would help current task

---

## 7. Repository Structure

### Complete Directory Layout

```
your-org/
├── octocat-supply-platform/               # Master orchestrator repo
│   ├── .github/
│   │   ├── copilot-instructions.md        # General repo instructions
│   │   ├── instructions/
│   │   │   ├── api.instructions.md        # API review guidance
│   │   │   ├── frontend.instructions.md   # Frontend review guidance
│   │   │   └── database.instructions.md   # DB migration guidance
│   │   └── skills/
│   │       ├── multi-repo-orchestration/
│   │       │   ├── SKILL.md
│   │       │   └── dependencies.json
│   │       ├── cross-repo-pr-linking/
│   │       │   └── SKILL.md
│   │       └── architecture-context/
│   │           └── SKILL.md
│   ├── architecture/
│   │   ├── README.md                      # System overview + ERD
│   │   ├── entity-model.md                # Entity relationships
│   │   └── components/
│   ├── specs/
│   │   ├── api/                           # OpenAPI / Swagger contracts
│   │   └── features/                      # Feature requirement docs
│   ├── docs/
│   │   ├── architecture.md
│   │   ├── build.md
│   │   ├── deployment.md
│   │   └── sqlite-integration.md
│   └── infra/
│       ├── container-apps.bicep           # Azure deployment
│       └── docker-compose.yml
│
├── octocat-supply-web/                    # Frontend repo (React + Vite + Tailwind)
│   ├── .github/
│   │   ├── copilot-instructions.md
│   │   ├── instructions/
│   │   │   └── frontend.instructions.md   # React/Tailwind review rules
│   │   └── skills/
│   │       └── report-to-master/
│   │           └── SKILL.md
│   ├── public/
│   │   └── runtime-config.js
│   └── src/
│       ├── App.tsx                        # Root component + routes
│       ├── main.tsx                       # Vite entry point
│       ├── index.css                      # Tailwind imports
│       ├── api/
│       │   └── config.ts                  # API base URL config
│       ├── components/
│       │   ├── Navigation.tsx
│       │   ├── Welcome.tsx
│       │   ├── About.tsx
│       │   ├── Footer.tsx
│       │   ├── Login.tsx
│       │   ├── admin/
│       │   │   └── AdminProducts.tsx       # Admin CRUD for products
│       │   └── entity/
│       │       └── product/               # Entity-based component tree
│       └── context/
│           ├── AuthContext.tsx             # Authentication provider
│           └── ThemeContext.tsx            # Dark mode / theming
│
├── octocat-supply-api/                    # API repo (Express + SQLite)
│   ├── .github/
│   │   ├── copilot-instructions.md
│   │   ├── instructions/
│   │   │   ├── api.instructions.md        # Express/repo review rules
│   │   │   └── database.instructions.md   # Migration review rules
│   │   └── skills/
│   │       ├── report-to-master/
│   │       │   └── SKILL.md
│   │       └── api-endpoint/
│   │           └── SKILL.md               # CRUD endpoint generation
│   ├── database/
│   │   ├── migrations/
│   │   │   ├── 001_init.sql               # Core tables
│   │   │   └── 002_add_supplier_status_fields.sql
│   │   └── seed/
│   │       ├── 001_suppliers.sql
│   │       ├── 002_headquarters.sql
│   │       ├── 003_branches.sql
│   │       └── 004_products.sql
│   └── src/
│       ├── index.ts                       # Express app bootstrap
│       ├── init-db.ts                     # DB initialization
│       ├── seedData.ts                    # Seed runner
│       ├── db/
│       │   ├── config.ts                  # DB configuration
│       │   ├── migrate.ts                 # Migration runner
│       │   ├── seed.ts                    # Seed executor
│       │   └── sqlite.ts                  # SQLite connection
│       ├── models/                        # TypeScript entity interfaces
│       │   ├── supplier.ts
│       │   ├── product.ts
│       │   ├── order.ts
│       │   ├── orderDetail.ts
│       │   ├── delivery.ts
│       │   ├── orderDetailDelivery.ts
│       │   ├── headquarters.ts
│       │   └── branch.ts
│       ├── repositories/                  # Data access layer
│       │   ├── suppliersRepo.ts
│       │   ├── productsRepo.ts
│       │   ├── ordersRepo.ts
│       │   ├── orderDetailsRepo.ts
│       │   ├── deliveriesRepo.ts
│       │   ├── orderDetailDeliveriesRepo.ts
│       │   ├── headquartersRepo.ts
│       │   └── branchesRepo.ts
│       ├── routes/                        # Express route handlers
│       │   ├── supplier.ts                # /api/suppliers
│       │   ├── product.ts                 # /api/products
│       │   ├── order.ts                   # /api/orders
│       │   ├── orderDetail.ts             # /api/order-details
│       │   ├── delivery.ts                # /api/deliveries
│       │   ├── orderDetailDelivery.ts     # /api/order-detail-deliveries
│       │   ├── headquarters.ts            # /api/headquarters
│       │   └── branch.ts                  # /api/branches
│       └── utils/
│           ├── errors.ts                  # Custom error classes
│           └── sql.ts                     # SQL helpers, camelCase mapping
│
└── octocat-supply-common/                 # Shared types repo
    ├── .github/
    │   ├── copilot-instructions.md
    │   └── skills/
    │       └── report-to-master/
    │           └── SKILL.md
    └── src/
        ├── models/                            # Shared entity interfaces
        │   ├── supplier.ts
        │   ├── product.ts
        │   ├── order.ts
        │   └── ...                            # Mirror of API models
        ├── dto/                               # Data transfer objects
        └── utils/                             # Shared utility functions
```

### Master Repo Details (`octocat-supply-platform`)

The master orchestrator repository is the **single source of truth** for architecture, specifications, and cross-repo coordination. It does not contain application source code; instead it holds:

| Concern | Contents |
|---------|----------|
| **Architecture** | System overview, ERD, entity-model docs (`architecture/`) |
| **Specifications** | OpenAPI contracts and feature requirement docs (`specs/`) |
| **Documentation** | Build, deployment, SQLite integration guides (`docs/`) |
| **Infrastructure** | Bicep templates, Docker Compose, CI/CD workflows (`infra/`) |
| **Agent Skills** | Orchestration, PR-linking, and architecture-context skills (`.github/skills/`) |
| **Copilot Instructions** | General repo guidelines + path-scoped API / frontend / database instructions (`.github/instructions/`) |

**Key responsibilities:**

1. **Issue orchestration** — When a feature issue is assigned to `@copilot`, the `multi-repo-orchestration` skill analyzes the request and spawns child issues in `octocat-supply-common`, `octocat-supply-api`, and/or `octocat-supply-web`.
2. **Traceability** — The `cross-repo-pr-linking` skill tracks PRs across repos and maintains a live status table on the master issue.
3. **Architecture context** — The `architecture-context` skill provides entity-model docs, API contracts, and component specs to Copilot agents working in dependent repos.
4. **Governance** — Copilot instructions and review guidance files (`.github/instructions/*.instructions.md`) define code-quality, security, and style expectations shared by all repos.

### Frontend Repo Details (`octocat-supply-web`)

| Aspect | Detail |
|--------|--------|
| **Framework** | React 18+ with TypeScript |
| **Build tool** | Vite |
| **Styling** | Tailwind CSS (utility-first; dark mode via `ThemeContext`) |
| **Routing** | React Router v7 — routes: `/`, `/about`, `/products`, `/login`, `/admin/products` |
| **Data fetching** | React Query for server-state cache |
| **Auth** | `AuthContext` provider wrapping the app |
| **Key components** | `Navigation`, `Welcome`, `About`, `Footer`, `Login`, `AdminProducts`, `entity/product/*` |
| **Config** | API base URL via `src/api/config.ts`; runtime config in `public/runtime-config.js` |
| **Testing** | Playwright for E2E; React Testing Library for component tests |
| **Skills** | `report-to-master` — links PRs back to the master issue |

### API Repo Details (`octocat-supply-api`)

| Aspect | Detail |
|--------|--------|
| **Framework** | Express.js with TypeScript |
| **Database** | SQLite (file `api/data/app.db`; in-memory for tests) |
| **Pattern** | Repository classes (`*Repo.ts`) for data access |
| **Entities** | `Supplier`, `Product`, `Order`, `OrderDetail`, `Delivery`, `OrderDetailDelivery`, `Headquarters`, `Branch` |
| **Naming** | camelCase in TypeScript models ↔ snake_case in SQL columns (mapped via `utils/sql.ts`) |
| **Docs** | Swagger/OpenAPI via JSDoc comments on route files |
| **Migrations** | Sequential SQL files in `database/migrations/`; tracked in a `migrations` table |
| **Seed data** | Ordered SQL scripts in `database/seed/` |
| **Error handling** | Custom error classes (`NotFound`, `Validation`, `Conflict`) → consistent HTTP status codes |
| **Testing** | Vitest for unit tests (`*.test.ts` alongside repos/routes) |
| **Skills** | `report-to-master` + `api-endpoint` (CRUD endpoint scaffolding) |

---

## 8. Agent Skills Definitions

### 8.1 Multi-Repo Orchestration Skill

**Location:** `octocat-supply-platform/.github/skills/multi-repo-orchestration/SKILL.md`

```markdown
---
name: multi-repo-orchestration
description: Orchestrates OctoCAT Supply Chain feature development across multiple repositories. Use this when assigned an issue that requires changes in the React frontend, Express API, or shared types repos.
---

# Multi-Repo Orchestration

When you are assigned an issue in this repository that requires work across multiple codebases, follow this process using tools from the GitHub MCP Server:

## Step 1: Analyze the Feature Request

1. Read the issue description carefully
2. Load the dependency map from `dependencies.json` in this skill’s directory
3. Determine which repositories are affected based on the feature scope:
   - **Frontend work** (UI components, pages, routing, Tailwind styling) → `your-org/octocat-supply-web`
   - **API work** (Express routes, repository classes, models, migrations, Swagger docs) → `your-org/octocat-supply-api`
   - **Shared types/contracts** (TypeScript interfaces, DTOs, utilities) → `your-org/octocat-supply-common`

## Step 2: Create Linked Issues in Dependent Repos

For each affected repository, use the `create_issue` tool:

**Issue Title Format:**
```
[octocat-supply-platform#<ISSUE_NUMBER>] <Specific task description>
```

**Issue Body Template:**
```markdown
## Parent Issue
This task was spawned from your-org/octocat-supply-platform#<ISSUE_NUMBER>

## Task Description
<Specific requirements for this repository>

## Architecture Context
<Relevant specs from the master repo>

## OctoCAT Entity Model
Entities: Supplier, Product, Order, OrderDetail, Delivery, OrderDetailDelivery, Headquarters, Branch

## Integration Notes
- Depends on: <list any cross-repo dependencies>
- Provides: <what this repo contributes to the feature>

---
*Auto-generated by Copilot multi-repo orchestration*
```

**Assignees:** Always assign `copilot` to each spawned issue.

## Step 3: Update the Master Issue

After creating all dependent issues, use the `add_issue_comment` tool to update the master issue:

```markdown
## 🚀 Multi-Repo Tasks Spawned

| Repository | Issue | Scope |
|------------|-------|-------|
| octocat-supply-common | your-org/octocat-supply-common#XXX | Shared types/contracts |
| octocat-supply-api | your-org/octocat-supply-api#XXX | Express API endpoints + SQLite migrations |
| octocat-supply-web | your-org/octocat-supply-web#XXX | React UI components + routes |

### Recommended Integration Order
1. ✅ `octocat-supply-common` - Shared TypeScript interfaces (no dependencies)
2. ⏳ `octocat-supply-api` - Express routes + repository classes (depends on common)
3. ⏳ `octocat-supply-web` - React components + Tailwind styling (depends on api, common)

### Tracking
I will monitor progress and update this issue when PRs are created.
```

## Step 4: Monitor Progress

Periodically use `search_issues` to check for PRs that reference this master issue, and update the tracking comment accordingly.
```

### 8.2 Dependencies Configuration

**Location:** `octocat-supply-platform/.github/skills/multi-repo-orchestration/dependencies.json`

```json
{
  "repositories": {
    "frontend": {
      "owner": "your-org",
      "repo": "octocat-supply-web",
      "scope": ["ui", "components", "pages", "frontend", "client", "web", "tailwind", "react", "routing"],
      "stack": "React 18+ / Vite / Tailwind CSS / React Router v7 / React Query",
      "integrationOrder": 3
    },
    "backend": {
      "owner": "your-org",
      "repo": "octocat-supply-api",
      "scope": ["api", "endpoints", "database", "backend", "server", "service", "migration", "repository", "swagger"],
      "stack": "Express.js / SQLite / Repository pattern / Swagger-OpenAPI",
      "integrationOrder": 2
    },
    "shared": {
      "owner": "your-org",
      "repo": "octocat-supply-common",
      "scope": ["types", "contracts", "utilities", "shared", "common", "models", "dto"],
      "stack": "TypeScript / shared interfaces",
      "integrationOrder": 1
    }
  },
  "entities": [
    "Supplier", "Product", "Order", "OrderDetail",
    "Delivery", "OrderDetailDelivery", "Headquarters", "Branch"
  ],
  "featureMapping": {
    "supplier-management": ["shared", "backend", "frontend"],
    "product-catalog": ["shared", "backend", "frontend"],
    "order-processing": ["shared", "backend", "frontend"],
    "delivery-tracking": ["shared", "backend", "frontend"],
    "branch-management": ["shared", "backend", "frontend"],
    "headquarters-admin": ["shared", "backend", "frontend"],
    "api-endpoints": ["shared", "backend"],
    "ui-components": ["shared", "frontend"],
    "database-migration": ["backend"],
    "reporting-dashboard": ["backend", "frontend"]
  }
}
```

### 8.3 Cross-Repo PR Linking Skill

**Location:** `octocat-supply-platform/.github/skills/cross-repo-pr-linking/SKILL.md`

```markdown
---
name: cross-repo-pr-linking
description: Links pull requests from dependent repositories back to master issues. Use this when checking PR status or when notified about PRs in octocat-supply-web, octocat-supply-api, or octocat-supply-common.
---

# Cross-Repo PR Linking

When PRs are created in dependent repositories that reference a master issue, follow this process to maintain traceability:

## Detecting Related PRs

Use the `search_issues` tool with this query:
```
[octocat-supply-platform#<ISSUE_NUMBER>] is:pr
```

Search across all dependent repos:
- `your-org/octocat-supply-web`
- `your-org/octocat-supply-api`
- `your-org/octocat-supply-common`

## Updating Master Issue

When you find related PRs, use `add_issue_comment` to update the master issue:

```markdown
## 📦 PR Status Update

| Repository | PR | Status | CI |
|------------|-----|--------|-----|
| octocat-supply-common | your-org/octocat-supply-common#XX | ✅ Merged | ✅ Passing |
| octocat-supply-api | your-org/octocat-supply-api#XX | 🔄 Open | ✅ Passing |
| octocat-supply-web | your-org/octocat-supply-web#XX | 🔄 Open | ⏳ Running |

### Next Steps
- [ ] Review octocat-supply-api PR (Express routes + repository classes)
- [ ] Review octocat-supply-web PR after API merges (React components + Tailwind)
- [ ] Final integration testing
```

## Status Icons
- 🔄 Open - PR is open and awaiting review
- ✅ Merged - PR has been merged
- ❌ Closed - PR was closed without merging
- ⏳ Draft - PR is in draft state
```

### 8.4 Architecture Context Skill

**Location:** `octocat-supply-platform/.github/skills/architecture-context/SKILL.md`

```markdown
---
name: architecture-context
description: Provides OctoCAT Supply Chain architecture documentation and specifications. Use this when you need to understand the entity model (Suppliers, Products, Orders, etc.), component relationships, or API contracts.
---

# Architecture Context Provider

When implementing features that span multiple repositories, use this skill to gather relevant architecture context.

## Fetching Architecture Docs

Use the `get_file_contents` tool to retrieve:

1. **System Overview**: `your-org/octocat-supply-platform/architecture/README.md`
2. **Entity Model**: `your-org/octocat-supply-platform/architecture/entity-model.md`
3. **Component Specs**: `your-org/octocat-supply-platform/architecture/components/`
4. **API Contracts**: `your-org/octocat-supply-platform/specs/api/`
5. **Feature Specs**: `your-org/octocat-supply-platform/specs/features/`
6. **SQLite Integration**: `your-org/octocat-supply-platform/docs/sqlite-integration.md`

## Including Context in Spawned Issues

When creating issues in dependent repos, include relevant excerpts:

```markdown
## Architecture Context

### From System Overview
<relevant excerpt>

### Entity Model
Entities: Supplier, Product, Order, OrderDetail, Delivery, OrderDetailDelivery, Headquarters, Branch
Relationships: Headquarters → Branch → Order → OrderDetail → Product; Supplier → Delivery → OrderDetailDelivery

### API Contract
<relevant Swagger/OpenAPI spec>

### Component Requirements
<relevant component spec>
```

This ensures Copilot agents in dependent repos have the context they need without having to fetch it themselves.
```

### 8.5 Report to Master Skill (Dependent Repos)

**Location:** `octocat-supply-web/.github/skills/report-to-master/SKILL.md` (same for octocat-supply-api, octocat-supply-common)

```markdown
---
name: report-to-master
description: Reports progress back to the master repository (octocat-supply-platform). Use this when creating PRs for issues that reference octocat-supply-platform, or when completing tasks spawned from the master repo.
---

# Report to Master Repository

When working on issues that were spawned from `octocat-supply-platform`, follow this process to maintain traceability:

## Detecting Master Issue Reference

Look for issue titles or bodies containing:
- `[octocat-supply-platform#<NUMBER>]`
- `your-org/octocat-supply-platform#<NUMBER>`
- `Spawned from octocat-supply-platform`

Extract the master issue number from these references.

## Creating PRs with Proper Linking

When creating a PR, use this title format:
```
[octocat-supply-platform#<ISSUE_NUMBER>] <Descriptive title>
```

In the PR body, include:
```markdown
## Related Issues
- Master: your-org/octocat-supply-platform#<ISSUE_NUMBER>
- This repo: #<LOCAL_ISSUE_NUMBER>

## Changes
<Description of changes>

## Cross-Repo Dependencies
- Depends on: <list any PRs this depends on>
- Required by: <list any PRs that depend on this>
```

## Notifying Master Repository

After creating the PR, use the `add_issue_comment` tool to comment on the master issue:

```markdown
## 🔗 PR Created in <REPO_NAME>

**PR:** your-org/<REPO_NAME>#<PR_NUMBER>
**Status:** Ready for review
**CI:** Pending

### Summary
<Brief description of what this PR implements>

### Integration Notes
<Any notes about dependencies or integration order>
```

## On PR Merge

When your PR is merged, update the master issue:

```markdown
## ✅ PR Merged in <REPO_NAME>

**PR:** your-org/<REPO_NAME>#<PR_NUMBER> has been merged.

<REPO_NAME> component is now complete for this feature.
```
```

### 8.6 Dependent Repos Custom Instructions

**Location:** `octocat-supply-web/.github/copilot-instructions.md` (similar for octocat-supply-api, octocat-supply-common)

```markdown
# OctoCAT Supply Chain – Frontend Repository Instructions

This repository is the **React + Vite + Tailwind CSS** frontend for the OctoCAT Supply Chain Management application, part of the `octocat-supply-platform` ecosystem.

## Tech Stack

- **Framework:** React 18+ with TypeScript
- **Build:** Vite
- **Styling:** Tailwind CSS (utility classes, dark mode support via ThemeContext)
- **Routing:** React Router v7
- **Data Fetching:** React Query for server cache
- **Auth:** AuthContext provider
- **Key views:** Welcome, About, Products, Login, Admin Products

## Multi-Repo Workflow

When assigned an issue that references `octocat-supply-platform#<number>`:

1. This issue was spawned by the orchestrator in the master repo
2. Implement the required changes following the task description
3. Use the `report-to-master` skill to create properly linked PRs
4. The master issue will be automatically updated with your progress

## PR Conventions

- **Title Format:** `[octocat-supply-platform#<issue>] <description>`
- **Always link** to both the master issue and local issue
- **Include integration notes** if your changes depend on other repos

## Frontend Guidelines

- Semantic HTML, proper ARIA labels, focus management
- Tailwind utility classes; extract repeated patterns into small components
- Components < 150 LOC; split data + presentation
- Use React Query for data fetching (not bare useEffect)
- Avoid `any`; type API responses with shared DTOs
- Responsive: test at mobile (≤640px), md (~768px), lg (≥1024px)

## Architecture Reference

For architecture context, refer to:
- `your-org/octocat-supply-platform/architecture/`
- `your-org/octocat-supply-platform/specs/`
- `your-org/octocat-supply-platform/docs/architecture.md`
```

---

**Location:** `octocat-supply-api/.github/copilot-instructions.md`

```markdown
# OctoCAT Supply Chain – API Repository Instructions

This repository is the **Express.js + SQLite** REST API for the OctoCAT Supply Chain Management application, part of the `octocat-supply-platform` ecosystem.

## Tech Stack

- **Framework:** Express.js with TypeScript
- **Database:** SQLite (file-based; in-memory for tests)
- **Pattern:** Repository classes for data access
- **Documentation:** Swagger/OpenAPI via JSDoc comments on routes
- **Entities:** Supplier, Product, Order, OrderDetail, Delivery, OrderDetailDelivery, Headquarters, Branch
- **Naming:** camelCase in TypeScript models, snake_case in SQL columns

## Multi-Repo Workflow

When assigned an issue that references `octocat-supply-platform#<number>`:

1. This issue was spawned by the orchestrator in the master repo
2. Implement the required changes following the task description
3. Use the `report-to-master` skill to create properly linked PRs
4. The master issue will be automatically updated with your progress

## PR Conventions

- **Title Format:** `[octocat-supply-platform#<issue>] <description>`
- **Always link** to both the master issue and local issue
- **Include integration notes** if your changes depend on other repos

## API Guidelines

- Thin routes → logic in repository classes
- Parameterized SQL always; never interpolate user input
- Custom error classes (NotFound, Validation, Conflict) → consistent HTTP status codes
- New schema changes require sequential migration files; never edit prior migrations
- Update Swagger docs when adding/modifying endpoints
- Unit tests for repository methods; integration tests for routes (supertest)

## Architecture Reference

For architecture context, refer to:
- `your-org/octocat-supply-platform/architecture/`
- `your-org/octocat-supply-platform/specs/`
- `your-org/octocat-supply-platform/docs/sqlite-integration.md`
```

---

## 9. Workflow Sequence

### End-to-End Flow

| Step | Action | Actor | Skill Used | MCP Tools |
|------|--------|-------|------------|-----------|
| 1 | Create feature issue in `octocat-supply-platform` | Human | - | - |
| 2 | Assign issue to `@copilot` | Human | - | - |
| 3 | Analyze feature requirements | Copilot | `multi-repo-orchestration` | `get_file_contents` |
| 4 | Read dependency map | Copilot | `multi-repo-orchestration` | `get_file_contents` |
| 5 | Create issue in `octocat-supply-common` | Copilot | `multi-repo-orchestration` | `create_issue` |
| 6 | Create issue in `octocat-supply-api` | Copilot | `multi-repo-orchestration` | `create_issue` |
| 7 | Create issue in `octocat-supply-web` | Copilot | `multi-repo-orchestration` | `create_issue` |
| 8 | Update master issue with tracking table | Copilot | `multi-repo-orchestration` | `add_issue_comment` |
| 9 | Copilot agent activates in `octocat-supply-common` | Copilot | - | - |
| 10 | Fetch architecture context | Copilot | `report-to-master` | `get_file_contents` |
| 11 | Implement shared types | Copilot | - | - |
| 12 | Create PR in `octocat-supply-common` | Copilot | `report-to-master` | `create_pull_request` |
| 13 | Notify master issue | Copilot | `report-to-master` | `add_issue_comment` |
| 14 | (Repeat 9–13 for `octocat-supply-api`: Express routes, repos, migrations) | Copilot | `report-to-master` | Various |
| 15 | (Repeat 9–13 for `octocat-supply-web`: React components, Tailwind UI) | Copilot | `report-to-master` | Various |
| 16 | Check all PR statuses | Copilot | `cross-repo-pr-linking` | `search_issues` |
| 17 | Update master issue with final status | Copilot | `cross-repo-pr-linking` | `add_issue_comment` |
| 18 | Human reviews all PRs | Human | - | - |

### Sequence Diagram

```
Human          Master Repo              supply-common     supply-api       supply-web
  │                 │                          │               │                │
  │──Create Issue──▶│                          │               │                │
  │──Assign @copilot▶│                         │               │                │
  │                 │                          │               │                │
  │                 │──Create Issue (types)────▶│               │                │
  │                 │──Create Issue (API)──────────────────────▶│                │
  │                 │──Create Issue (React UI)─────────────────────────────────▶│
  │                 │                          │               │                │
  │                 │◀──PR Created─────────────│               │                │
  │                 │◀──PR Created (Express)───────────────────│                │
  │                 │◀──PR Created (React)─────────────────────────────────────│
  │                 │                          │               │                │
  │◀──PR Links──────│                          │               │                │
  │                 │                          │               │                │
  │──Review PRs────▶│                          │               │                │
  │                 │                          │               │                │
```

---

## 10. Implementation Guide

### Step 1: Create Master Repository Structure

```bash
# Create the master orchestrator repo
mkdir -p octocat-supply-platform/.github/skills/multi-repo-orchestration
mkdir -p octocat-supply-platform/.github/skills/cross-repo-pr-linking
mkdir -p octocat-supply-platform/.github/skills/architecture-context
mkdir -p octocat-supply-platform/.github/instructions
mkdir -p octocat-supply-platform/architecture/components
mkdir -p octocat-supply-platform/specs/api
mkdir -p octocat-supply-platform/specs/features
mkdir -p octocat-supply-platform/docs
mkdir -p octocat-supply-platform/infra
```

### Step 2: Add Skill Files

Copy the SKILL.md files from Section 8 into their respective directories.

### Step 3: Configure Dependent Repositories

```bash
# Frontend repo (React + Vite + Tailwind)
mkdir -p octocat-supply-web/.github/skills/report-to-master
mkdir -p octocat-supply-web/.github/instructions
# Add SKILL.md, copilot-instructions.md, frontend.instructions.md

# API repo (Express + SQLite)
mkdir -p octocat-supply-api/.github/skills/report-to-master
mkdir -p octocat-supply-api/.github/skills/api-endpoint
mkdir -p octocat-supply-api/.github/instructions
# Add SKILL.md, copilot-instructions.md, api.instructions.md, database.instructions.md

# Shared types repo
mkdir -p octocat-supply-common/.github/skills/report-to-master
# Add SKILL.md, copilot-instructions.md
```

### Step 4: Update dependencies.json

Replace `your-org` with your actual GitHub organization name in:
- `dependencies.json`
- All SKILL.md files

### Step 5: Enable Copilot Coding Agent

1. Go to each repository's Settings
2. Navigate to Copilot → Coding Agent
3. Enable the feature
4. Configure allowed tools (GitHub MCP Server)

### Step 6: Test the Workflow

1. Create a test issue in `octocat-supply-platform`:
   ```
   Title: [Test] Add supplier status dashboard
   Body: Add a new dashboard showing supplier active/inactive status across the frontend and API
   ```
2. Assign to `@copilot`
3. Monitor the spawned issues and PRs

---

## 11. Success Metrics

### Key Performance Indicators

| Metric | Target | Measurement |
|--------|--------|-------------|
| **Issue Spawn Rate** | 100% of multi-repo features | Count of spawned issues / multi-repo issues |
| **PR Link Rate** | 100% of spawned PRs | PRs linked to master / total spawned PRs |
| **Time to First PR** | < 30 minutes | Time from issue assignment to first PR |
| **Human Intervention Rate** | < 10% | Orchestrations requiring human help |
| **Integration Success Rate** | > 90% | PRs that merge without conflicts |

### Tracking Dashboard

Monitor these GitHub metrics:
- Issues with `multi-repo` label
- PRs with `[octocat-supply-platform#X]` title pattern
- Comment activity on master issues
- CI pass rates on spawned PRs

---

## 12. Dependencies & Prerequisites

### Required

| Dependency | Version | Purpose |
|------------|---------|---------|
| GitHub Copilot | Pro/Pro+/Business/Enterprise | Agent runtime |
| GitHub MCP Server | Latest | Cross-repo operations |
| Repository write access | - | Create issues/PRs/comments |

### Configuration Requirements

- [ ] Copilot coding agent enabled in all repositories
- [ ] GitHub MCP Server configured with appropriate token
- [ ] Token has `repo` scope for all dependent repositories
- [ ] Skills directories created in all repositories

### Permissions Matrix

| Action | Required Permission |
|--------|---------------------|
| Create issues | `issues: write` |
| Create PRs | `pull_requests: write` |
| Add comments | `issues: write` |
| Read files | `contents: read` |
| Search issues | `issues: read` |

---

## 13. Risks & Mitigations

### Risk Assessment

| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| **Rate limiting** | Medium | High | Batch operations, use search before create |
| **Token expiration** | Low | High | Set up token rotation, monitor expiry |
| **Circular dependencies** | Low | Medium | Validate dependency graph at spawn time |
| **Lost context** | Medium | Medium | Include full context in spawned issues |
| **CI failures blocking PRs** | Medium | Low | Include CI status in updates, suggest fixes |
| **Skill not triggering** | Medium | Medium | Test descriptions, use explicit triggers |

### Fallback Procedures

1. **If spawning fails**: Copilot comments on master issue with error details
2. **If PR linking fails**: Manual linking with standard GitHub references
3. **If agent times out**: Human can re-trigger by reassigning issue

---

## Appendix A: Glossary

| Term | Definition |
|------|------------|
| **Agent Skill** | A folder of instructions that Copilot loads for specialized tasks |
| **MCP** | Model Context Protocol — standard for AI tool integration |
| **Master Repo** | `octocat-supply-platform` — holds architecture specs, docs, infra, and orchestration skills |
| **Frontend Repo** | `octocat-supply-web` — React 18 + Vite + Tailwind CSS single-page application |
| **API Repo** | `octocat-supply-api` — Express.js + SQLite REST API with repository pattern |
| **Shared Repo** | `octocat-supply-common` — shared TypeScript interfaces, DTOs, and utilities |
| **Dependent Repo** | A repository containing implementation code (frontend / API / shared) |
| **Spawned Issue** | An issue created by Copilot in a dependent repo |
| **Entity** | A domain object in the OctoCAT model (Supplier, Product, Order, etc.) |

---

## Appendix B: References

- [About Agent Skills - GitHub Docs](https://docs.github.com/en/enterprise-cloud@latest/copilot/concepts/agents/about-agent-skills)
- [Agent Skills Open Standard](https://github.com/agentskills/agentskills)
- [Anthropic Skills Collection](https://github.com/anthropics/skills)
- [Awesome Copilot](https://github.com/github/awesome-copilot)
- [GitHub MCP Server](https://github.com/modelcontextprotocol/servers)

---

## Appendix C: Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | 2026-02-05 | Initial PRD |
| 2.0 | 2026-02-06 | Updated to OctoCAT Supply Chain repos (`octocat-supply-platform`, `octocat-supply-web`, `octocat-supply-api`, `octocat-supply-common`). Added concrete frontend (React + Vite + Tailwind) and API (Express + SQLite) details. Added master orchestrator repo with architecture, docs, and infra. Added entity model, per-repo custom instructions, and supply-chain–specific feature mappings. |

---
