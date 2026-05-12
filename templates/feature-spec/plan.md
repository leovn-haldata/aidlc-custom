# Plan: <change-id>

## Change Reference
- **Design**: [design.md](./design.md)
- **Mode**: feature-spec

## Component Design

_Which components are involved and how they interact._

### Component: _name_
- **Responsibility**: _what it does_
- **Interfaces**: _what it exposes_
- **Dependencies**: _what it depends on_

## Architecture Decisions

| Decision | Options Considered | Chosen | Rationale | Consequences |
|----------|--------------------|--------|-----------|-------------|
| _Decision 1_ | _A, B_ | _A_ | _Why_ | _Trade-offs_ |

## Data Flow

_How data moves through the system for this feature._

1. _Actor_ sends _request_ to _Component A_
2. _Component A_ validates and calls _Component B_
3. _Component B_ persists data and dispatches _event_
4. _Response_ returned to _Actor_

## Dependency Analysis

| Module | Type | Notes |
|--------|------|-------|
| `path/to/module` | Existing / New | _dependency description_ |

## Risk Mitigation

| Risk | Mitigation |
|------|-----------|
| _Technical risk 1_ | _How the plan addresses it_ |

## Repo Boundaries (cross-repo only)

> Remove this section if the change is single-repo.

### <repo-name>
- **Components**: _list_
- **Contracts at boundary**: _API shape / event schema / shared type_

### <repo-name>
- **Components**: _list_
- **Contracts at boundary**: _API shape / event schema / shared type_
