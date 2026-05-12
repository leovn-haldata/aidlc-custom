# Example: Multi-Repo Feature - CSV User Import

This example demonstrates the full AIDLC workflow for a cross-repo feature that spans backend, API, and frontend repositories.

## Scenario

**Feature Request**: "Add bulk user import via CSV upload for admins"

**Repos involved**:
- `tv-backend`: Batch processing, user creation, notification jobs
- `tv-api`: Upload endpoint, import status API, validation
- `tv-app`: Upload UI, progress display, user list integration

## Hub Repo Structure

```
sdd-repo/                           ← Orchestration hub
├── .aidlc/config.md                ← Multi-repo configuration
├── aidlc-docs/changes/active/
│   └── csv-user-import/
│       ├── intake-note.md
│       ├── proposal.md
│       ├── design.md
│       ├── plan.md                 ← Repo boundaries defined here
│       ├── contracts/              ← Frozen contracts
│       │   ├── api-contracts.md
│       │   ├── event-contracts.md
│       │   └── schema-contracts.md
│       ├── tasks.md                ← [REPO: xxx] tagged tasks
│       ├── build-summary.md        ← Links to all 3 PRs
│       └── review-report.md
└── templates/
```

## Code Repos Structure

```
tv-backend/.aidlc/link.md → ../sdd-repo
tv-api/.aidlc/link.md     → ../sdd-repo  
tv-app/.aidlc/link.md     → ../sdd-repo
```

## Workflow Steps

### Phase 1: Governance (Claude Code at sdd-repo/)

```bash
cd /path/to/sdd-repo
claude

> /aidlc-intake "Add bulk user import via CSV for admins. 
  Backend processes CSV, API handles upload/status, Frontend shows UI"
  
  Mode recommended: full-spec (cross-repo, contracts needed)
  
> /aidlc-spec csv-user-import
  
  Creates: intake-note.md, proposal.md, design.md
  
> /aidlc-confirm csv-user-import

  Status: CONFIRMED
  
> /aidlc-plan csv-user-import

  Creates: plan.md with Repo Boundaries
  Creates: contracts/ folder (draft)
  
  Human decision: FREEZE CONTRACTS? → Yes
  
  All contracts marked Status: frozen
  
> /aidlc-tasks csv-user-import

  Contract guard: ✓ All contracts frozen
  
  Creates: tasks.md with [REPO: tv-backend], [REPO: tv-api], [REPO: tv-app] tags
```

### Phase 2: Implementation (Cursor per repo)

#### tv-backend implementation

```bash
# Open tv-backend in Cursor
cd /path/to/tv-backend
cursor .

> /aidlc-build csv-user-import

  Agent reads .aidlc/link.md → finds hub at ../sdd-repo
  Loads tasks.md → filters [REPO: tv-backend] tasks only:
  
  - Task 1: Create CSV batch processor job
  - Task 2: Add user creation logic with validation
  - Task 3: Emit user-import-progress events
  - Task 4: Handle notification job
  
  TDD cycle: tests → implementation → commit
  Branch: feature/csv-user-import
  PR created: tv-backend#123
```

#### tv-api implementation

```bash
# Open tv-api in Cursor  
cd /path/to/tv-api
cursor .

> /aidlc-build csv-user-import

  Filters [REPO: tv-api] tasks:
  
  - Task 5: POST /imports/csv endpoint (multipart upload)
  - Task 6: GET /imports/{id}/status endpoint
  - Task 7: Consume user-import-progress events
  - Task 8: Input validation using frozen schemas
  
  Contract references:
  - Contract: contracts/api-contracts.md#POST /imports/csv
  - Contract: contracts/event-contracts.md#user-import-progress
  
  Branch: feature/csv-user-import
  PR created: tv-api#456
```

#### tv-app implementation

```bash
# Open tv-app in Cursor
cd /path/to/tv-app
cursor .

> /aidlc-build csv-user-import

  Filters [REPO: tv-app] tasks:
  
  - Task 9: CSV upload form component
  - Task 10: Import progress display
  - Task 11: Integrate with user list page
  - Task 12: Error handling and retry UI
  
  Contract references:
  - Contract: contracts/api-contracts.md#POST /imports/csv
  - Contract: contracts/api-contracts.md#GET /imports/{id}/status
  
  Branch: feature/csv-user-import
  PR created: tv-app#789
```

### Phase 3: Review & Release (Claude Code at sdd-repo/)

```bash
cd /path/to/sdd-repo
claude

> /aidlc-review csv-user-import

  Reads build-summary.md → finds 3 PRs:
  - tv-backend#123: Batch processor + events
  - tv-api#456: Upload endpoints + validation  
  - tv-app#789: Upload UI + progress display
  
  Cross-repo contract compliance check:
  ✓ API endpoints match frozen contracts
  ✓ Event payloads match frozen contracts  
  ✓ Error codes consistent across repos
  
  Status: APPROVED for merge

> /aidlc-release csv-user-import

  Merge order from .aidlc/config.md:
  1. tv-backend → tv-api → tv-app
  
  Executes:
  1. Merge tv-backend#123 → verify CI passes
  2. Merge tv-api#456 → run integration tests  
  3. Merge tv-app#789 → E2E test pass
  
  Creates release notes linking all 3 PRs
  Moves spec to aidlc-docs/changes/released/
```

## Key Files Generated

### `.aidlc/config.md` (hub repo)

```markdown
## Project
- Name: TV System
- Type: multi-repo
- Hub: /path/to/sdd-repo

## Repos  
| Name       | Path          | Role              | Merge Order |
|------------|---------------|-------------------|-------------|
| tv-backend | ../tv-backend | Jobs + business   | 1           |
| tv-api     | ../tv-api     | REST API layer    | 2           |
| tv-app     | ../tv-app     | Frontend UI       | 3           |

## Merge Order
tv-backend → tv-api → tv-app
```

### `contracts/api-contracts.md` (excerpt)

```markdown
## POST /imports/csv
**Status**: frozen
**Provider**: tv-api
**Consumers**: tv-app

Request:
```
Content-Type: multipart/form-data
file: <CSV file>
notifyUsers: boolean
```

Response 202:
```json
{
  "importId": "imp_123456",
  "status": "queued"
}
```
```

### `tasks.md` (excerpt)

```markdown
## Phase 1: Backend Foundation [REPO: tv-backend]
- [x] Task 1: CSV batch processor @dev-alice [DONE] [REPO: tv-backend]
  - Files: jobs/csv_import_processor.py
  - Test: tests/jobs/test_csv_import.py
  - Contract: contracts/event-contracts.md#user-import-progress
  - Est: 4h

## Phase 2: API Layer [REPO: tv-api]  
- [x] Task 5: Upload endpoint @dev-bob [DONE] [REPO: tv-api]
  - Files: routes/imports.py, validators/csv.py
  - Test: tests/routes/test_imports.py
  - Contract: contracts/api-contracts.md#POST /imports/csv
  - Depends on: Task 1
  - Est: 3h

## Phase 3: Frontend [REPO: tv-app]
- [x] Task 9: Upload form @dev-charlie [DONE] [REPO: tv-app]  
  - Files: components/CsvUpload.tsx
  - Test: components/__tests__/CsvUpload.test.tsx
  - Contract: contracts/api-contracts.md#POST /imports/csv
  - Depends on: Task 5
  - Est: 2h
```

## Lessons Learned

1. **Contract-first prevented integration issues**: Freezing API contracts before implementation meant no surprises during integration
2. **Hub repo kept everyone aligned**: Single source of truth for progress, contracts, and decisions  
3. **Task tagging enabled parallel work**: Teams could work independently on their repo slices
4. **Merge order prevented breakage**: Lower-layer repos merged first, upper layers integrated against stable interfaces

This pattern scales well for teams with clear repo ownership and well-defined service boundaries.