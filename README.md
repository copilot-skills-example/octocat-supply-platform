# OctoCAT Supply – Platform (Master Orchestrator)

Central repository for the **OctoCAT Supply Chain Management** multi-repo system. This repo owns architecture docs, deployment infrastructure, the PRD, and Copilot Agent Skills that coordinate work across the child repos.

## Repository Map

| Repo | Purpose |
|------|---------|
| **octocat-supply-platform** (this) | Master orchestrator, docs, infra |
| [octocat-supply-web](https://github.com/octocat-supply/octocat-supply-web) | React + Vite + Tailwind frontend |
| [octocat-supply-api](https://github.com/octocat-supply/octocat-supply-api) | Express.js REST API (SQLite) |
| [octocat-supply-common](https://github.com/octocat-supply/octocat-supply-common) | Shared TypeScript types / models |

## Directory Structure

```
architecture/          # System architecture, entity model
specs/                 # API & feature specifications
docs/                  # Detailed docs (build, deployment, SQLite, etc.)
infra/                 # Bicep templates, container-apps config
.github/
  copilot-instructions.md
  skills/              # Copilot Agent Skills (multi-repo orchestration, cross-repo PR linking, architecture context)
  instructions/        # Path-scoped review guidance
```

## Copilot Agent Skills

| Skill | Trigger |
|-------|---------|
| **multi-repo-orchestration** | Issues labelled `feature:cross-repo` — spawns child issues to web/api/common repos |
| **cross-repo-pr-linking** | PRs in child repos — ensures master tracking issue is updated |
| **architecture-context** | Any prompt needing entity model or architecture context |

## Getting Started

```bash
# Clone all repos
gh repo clone octocat-supply/octocat-supply-platform
gh repo clone octocat-supply/octocat-supply-web
gh repo clone octocat-supply/octocat-supply-api
gh repo clone octocat-supply/octocat-supply-common
```

See [docs/build.md](docs/build.md) and [docs/deployment.md](docs/deployment.md) for build & deploy instructions.

## License

MIT
