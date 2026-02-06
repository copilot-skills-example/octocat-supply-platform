# Feature Specifications

This directory holds feature requirement documents for cross-cutting features that span multiple repositories.

## Current Features

### Cart Page (`cart-*.md`)
Shopping cart functionality spanning frontend and backend:
- **Repositories:** octocat-supply-api, octocat-supply-web
- **Status:** Specification complete, awaiting implementation
- **Parent Issue:** copilot-skills-example/octocat-supply-platform#1
- **Files:**
  - `cart-page.md` - Technical specification with phase-by-phase implementation details
  - `cart-orchestration-plan.md` - Multi-repo coordination and integration flow
  - `cart-issue-templates.md` - Ready-to-use issue descriptions for dependent repos

## How to Add a Feature Spec

1. Create a new markdown file: `<feature-name>.md`
2. Include:
   - **Summary** — What the feature does
   - **Affected repos** — Which of `octocat-supply-web`, `octocat-supply-api` are involved
   - **Entity changes** — Any new entities or modifications to existing ones
   - **API changes** — New or modified endpoints
   - **UI changes** — New pages, components, or modifications
   - **Migration requirements** — Database schema changes
   - **Acceptance criteria** — How to verify the feature works

## Using These Specifications

### For Platform Orchestrators
1. Review the orchestration plan to understand cross-repo dependencies
2. Use issue templates to spawn work items in dependent repositories
3. Monitor progress using the tracking guidelines

### For Repository Implementers
1. Read the feature specification for your repository's scope
2. Follow the entity model and API contracts defined in `/architecture/entity-model.md`
3. Reference the orchestration plan for integration points with other repos

## Example

See the multi-repo orchestration skill (`.github/skills/multi-repo-orchestration`) for how Copilot uses these specs when spawning issues in dependent repositories.
