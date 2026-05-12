# Plan: <change-id>

## Change Reference
- **Design**: [design.md](./design.md)
- **Data Model**: [data-model.md](./data-model.md)
- **Research**: [research.md](./research.md)
- **Mode**: full-spec

## Component Design

_Which components are involved and how they interact._

### Component: _name_
- **Responsibility**: _what it does_
- **Interfaces**: _what it exposes_
- **Dependencies**: _what it depends on_

## Domain Model

### Entities & Aggregates
| Entity | Type | Description |
|--------|------|-------------|
| _Entity1_ | Aggregate Root | _what it represents_ |
| _Entity2_ | Value Object | _what it represents_ |

### Domain Events
| Event | Producer | Consumer(s) | Payload (key fields) |
|-------|----------|-------------|----------------------|
| _EventName_ | _Component_ | _Component_ | _fields_ |

## Logical Design

_Layer interactions and patterns applied._

- **Pattern**: _Repository / Factory / Strategy / etc._
- **Layer flow**: _Controller → Service → Repository → Model_

## Architecture Decisions

| Decision | Options Considered | Chosen | Rationale | Consequences |
|----------|--------------------|--------|-----------|-------------|
| _Decision 1_ | _A, B, C_ | _B_ | _Why_ | _Trade-offs_ |

> Full ADRs in `aidlc-docs/design-artifacts/adrs/`

## Data Flow

_How data moves through the system._

1. _Actor_ sends _request_ to _Component A_
2. _Component A_ validates and calls _Service_
3. _Service_ calls _Repository_ → persists to DB
4. _Service_ dispatches _DomainEvent_
5. _Consumer_ handles event asynchronously
6. _Response_ returned to _Actor_

## Dependency Analysis

| Module | Type | Notes |
|--------|------|-------|
| `path/to/module` | Existing / New | _dependency description_ |

## Risk Mitigation

| Risk | Mitigation |
|------|-----------|
| _Technical risk 1_ | _How the plan addresses it_ |

## Cross-Artifact Consistency Check

| Artifact | Checked | Notes |
|----------|---------|-------|
| User stories → components | ✓ / ✗ | _gaps if any_ |
| Data entities → plan references | ✓ / ✗ | _gaps if any_ |
| API endpoints → service handlers | ✓ / ✗ | _gaps if any_ |
| NFRs → architecture decisions | ✓ / ✗ | _gaps if any_ |

## Repo Boundaries (required for cross-repo)

> Remove this section if the change is single-repo.

### <repo-name>
- **Components**: _list_
- **Contracts at boundary**: _event payload schema_
- **PR branch**: `feature/<change-id>-<repo>`
- **Merge order**: 1

### <repo-name>
- **Components**: _list_
- **Contracts at boundary**: _REST endpoint shape_
- **PR branch**: `feature/<change-id>-<repo>`
- **Merge order**: 2

> Shared types that must stay in sync across repos: _list them here_
