---
name: cross-repo-pr-linking
description: Links pull requests from dependent repositories back to master issues. Use this when checking PR status or when notified about PRs in octocat-supply-web or octocat-supply-api.
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

## Updating Master Issue

When you find related PRs, use `add_issue_comment` to update the master issue:

```markdown
## 📦 PR Status Update

| Repository | PR | Status | CI |
|------------|-----|--------|-----|
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
