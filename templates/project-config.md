# Project Configuration: <project-name>

## Project Identity
- **Project ID**: <short-id> (used as prefix for change IDs, e.g., `PRJ`)
- **Project Name**: <full name>
- **Repository**: <repo URL>
- **AIDLC SKILLS Version**: skills-vX.Y (from `aidlc-custom/`)

## Team Structure
- **PM / BrSE** (Green Lane): <name(s)>
- **Tech Lead** (Blue Lane): <name(s)>
- **Dev Team** (Yellow Lane): <name(s)>
- **DevOps / SRE**: <name(s)>

## Tech Stack
- **Backend**: <e.g., Python 3.12 + FastAPI>
- **Frontend**: <e.g., Next.js 14 + Tailwind CSS>
- **Database**: <e.g., PostgreSQL 16>
- **AI/ML**: <e.g., vLLM + Qwen3> (if applicable)
- **Infrastructure**: <e.g., Docker + GCP Cloud Run>

## Workflow Settings

### Default Spec Mode
- **Default**: light-spec (override per task at `/aidlc-intake`)

### Artifact Path
- **Base path**: `aidlc-docs/changes/`
- **Active**: `aidlc-docs/changes/active/`
- **Released**: `aidlc-docs/changes/released/`
- **Archived**: `aidlc-docs/changes/archived/`

### Issue Tracker
- **Platform**: GitHub Issues | Jira | Linear | Azure DevOps | _other_
- **Change ID format**: `<PREFIX>-<number>` (e.g., `PRJ-042`)
- **Mapping**: See `docs/issue-tracker-integration.md`

### Branch Convention
- **Strategy**: gitflow | trunk-based | GitHub flow
- **Patterns**:
  - Feature: `feature/<change-id>`
  - Fix: `fix/<change-id>`
  - Hotfix: `hotfix/<issue-id>`
  - Release: `pre-release/<version>`
  - Production: `master` or `main`

### CI/CD
- **Platform**: GitHub Actions | GitLab CI | Jenkins | CircleCI | _other_
- **Pipeline config**: See `docs/cicd-integration.md`

### Deploy Targets

| Environment | URL | Deploy Trigger | Approval Required |
|-------------|-----|----------------|-------------------|
| Development | <url> | Merge to `develop` | No |
| Staging | <url> | Merge to `pre-release/*` | Tech Lead |
| Production | <url> | Merge to `master` | Tech Lead + PM |

## Sprint Settings
- **Duration**: 2 weeks (adjust for maintenance: 3-4 weeks)
- **Sprint naming**: `YYYY-SNN` (e.g., `2026-S08`)
- **Velocity tracking**: See `docs/estimation-guide.md`

## Customizations

### Per-Folder Rules

| Folder | Rules | Stack |
|--------|-------|-------|
| `backend/` | `backend/rules/` | <language + framework> |
| `frontend/` | `frontend/rules/` | <language + framework> |
| `ai/` | `ai/rules/` | <ML framework> |
| `deployment/` | `deployment/rules/` | <IaC tool> |

### Project-Specific Commands
_List any custom commands added for this project (beyond standard AIDLC):_
- (none yet)
