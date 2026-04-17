# Design: <change-id>

<!-- LOCKED_BY: | SINCE: -->

## Change Reference
- **Proposal**: [proposal.md](./proposal.md)
- **Research**: [research.md](./research.md)
- **Mode**: full-spec

## System Architecture

_High-level architecture overview. Describe components and their responsibilities._

## Component Design

### Component A: _name_
- **Responsibility**: _what it does_
- **Interfaces**: _what it exposes_
- **Dependencies**: _what it depends on_

### Component B: _name_
- **Responsibility**: _what it does_
- **Interfaces**: _what it exposes_
- **Dependencies**: _what it depends on_

## Sequence Flows

### Flow 1: _scenario name_

_Describe the step-by-step interaction between components for this scenario._

1. _Actor_ sends _request_ to _Component A_
2. _Component A_ validates and calls _Component B_
3. _Component B_ persists data and returns _response_
4. _Component A_ returns _result_ to _Actor_

## Architecture Decisions (ADRs)

### ADR-001: _Decision Title_
- **Status**: Proposed / Accepted / Deprecated
- **Context**: _Why this decision is needed_
- **Options**: _A, B, C_
- **Decision**: _Chosen option_
- **Rationale**: _Why_
- **Consequences**: _Trade-offs_

## Impacted Modules

| Module | Change Type | Description | Team |
|--------|------------|-------------|------|
| `path/to/module` | New / Modify | _what changes_ | _team_ |

## Risks

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|
| _Risk 1_ | Low/Med/High | Low/Med/High | _Mitigation_ |

## Performance Considerations

_Expected load, response time targets, bottleneck analysis._

## Security Considerations

_New attack surface, auth changes, data sensitivity._

## Backward Compatibility & Migration

_Breaking changes? Migration strategy? Feature flags?_
