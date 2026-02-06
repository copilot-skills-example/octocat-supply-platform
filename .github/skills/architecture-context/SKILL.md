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
