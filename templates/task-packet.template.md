# <change-id> — <repo-name> Task Packet

## Metadata
- **Change ID**: <change-id>
- **Repo**: <repo-name>
- **Sprint**: <sprint-number>
- **Teams**: <team-name(s)>
- **Contracts frozen**: <YYYY-MM-DD or N/A>
- **Hub spec**: `aidlc-docs/changes/active/<change-id>/`

## Reading order
Load context in this order before starting:
1. `proposal.md` — business context and goals
2. `affected-docs.md` (if exists) — docs relevant to this repo only
3. `contracts/` — frozen contracts this repo implements or consumes
4. This packet — scope, tasks, and tests for this repo

## Repo scope
This repo is responsible for:
- <list responsibilities extracted from tasks.md [REPO: <repo-name>] tasks>

## Out of scope
This repo must NOT:
- Modify other repos' behavior
- Change a frozen contract without cross-team sign-off (see `docs/team-conflict-playbook.md` Scenario 3)
- Implement work tagged `[REPO: <other-repo>]`

## Contracts this repo implements or consumes

<!-- Reference specific sections from the frozen contracts, not the full file.
     Format: <contract file>#<section> — role (provider / consumer) -->

| Contract | Section | Role | Frozen date |
|----------|---------|------|-------------|
| `contracts/api-contracts.md` | `<METHOD> /path` | provider / consumer | <YYYY-MM-DD> |

## Affected local docs
<!-- Extracted from affected-docs.md — "Read first" section for this repo only -->
- `<path/to/doc.md>` — <why it's relevant>

## Tasks
<!-- Copy only tasks tagged [REPO: <repo-name>] from tasks.md. Preserve IDs and dependencies. -->

- [ ] Task X.X: <description> @unassigned [UNASSIGNED]
  - Files: `<path>`
  - Test: `<path>`
  - Depends on: <task ID or none>
  - Est: <hours>

## Tests required
<!-- Extracted from test-plan.md — only test cases relevant to this repo -->
- <test case description> → expected: <outcome>

## Definition of done
- [ ] All tasks above completed and marked `[DONE]`
- [ ] Unit tests pass, coverage ≥ 80%
- [ ] No contract drift (implementation matches frozen contracts)
- [ ] Build and lint pass
- [ ] PR created with title: `aidlc(<change-id>): <short description>`
- [ ] PR body includes build summary and coverage report

## Questions for spec owner
<!-- Any clarifications needed before starting — resolve before marking tasks IN_PROGRESS -->
- <question>
