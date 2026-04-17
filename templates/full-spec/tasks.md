# Tasks: <change-id>

## Change Reference
- **Design**: [design.md](./design.md)
- **Data Model**: [data-model.md](./data-model.md)
- **Contracts**: [contracts/](./contracts/)
- **Mode**: full-spec

## Metadata
- **Sprint**: _sprint-number_
- **Epic** (if multi-sprint): _epic-name_
- **Teams**: _team-a, team-b_
- **Sprint Commitment Date**: _YYYY-MM-DD_

## Phase 1: Data Layer (Team: _name_)

- [ ] Task 1.1: _Create DB migration_ @unassigned [UNASSIGNED]
  - Files: `migrations/...`
  - Depends on: none
  - Sprint: _sprint-number_
  - Est: _time_

- [ ] Task 1.2: _Implement data models_ @unassigned [UNASSIGNED] [P]
  - Files: `backend/models/...`
  - Test: `tests/models/...`
  - Depends on: Task 1.1
  - Est: _time_

## Phase 2: API / Service Layer (Team: _name_)

- [ ] Task 2.1: _Implement service logic_ @unassigned [UNASSIGNED]
  - Files: `backend/services/...`
  - Test: `tests/services/...`
  - Depends on: Task 1.2
  - Shared interface: _[DEP: Task 2.2] -- service method signatures_
  - Est: _time_

- [ ] Task 2.2: _Create API endpoints_ @unassigned [UNASSIGNED]
  - Files: `backend/api/...`
  - Test: `tests/api/...`
  - Depends on: Task 2.1
  - Shared interface: _[DEP: Task 2.1] -- consumes service methods, [DEP: Task 3.1] -- API contract for frontend_
  - Contract: `contracts/api-spec.yaml`
  - Est: _time_

## Phase 3: Frontend (Team: _name_)

- [ ] Task 3.1: _Build UI components_ @unassigned [UNASSIGNED]
  - Files: `frontend/components/...`
  - Test: `tests/components/...`
  - Depends on: Task 2.2 (API available)
  - Est: _time_

## Phase 4: Cross-Team Integration

- [ ] Task 4.1: _End-to-end integration test_ @unassigned [UNASSIGNED]
  - Depends on: Phase 2 + Phase 3 complete
  - Teams: _all teams_
  - Est: _time_

- [ ] Task 4.2: _Performance/load test_ @unassigned [UNASSIGNED]
  - Depends on: Task 4.1
  - Est: _time_

## Quality Checkpoints

- [ ] All unit tests pass (each team)
- [ ] Integration tests pass
- [ ] E2E tests pass
- [ ] Coverage >= 80%
- [ ] Build passes
- [ ] API contracts validated against spec
- [ ] Data model migration tested (up + down)
- [ ] Security review complete
