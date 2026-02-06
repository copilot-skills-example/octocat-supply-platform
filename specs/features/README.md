# Feature Specifications

This directory holds feature requirement documents for cross-cutting features that span multiple repositories.

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

## Example

See the multi-repo orchestration skill for how Copilot uses these specs when spawning issues in dependent repositories.
