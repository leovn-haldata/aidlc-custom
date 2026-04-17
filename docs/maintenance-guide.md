# Maintenance Mode Guide

## When Does Maintenance Mode Start?

A project enters maintenance mode when:
- Primary development is complete (no major features planned)
- The project is in stable production
- Team transitions from full dev team to skeleton crew
- Focus shifts from feature delivery to stability, security, and support

## Maintenance Activities

### Routine (Weekly / Bi-weekly)

| Activity | Frequency | Spec Mode | Owner |
|----------|-----------|-----------|-------|
| Dependency security audit (`npm audit`, `pip-audit`) | Weekly | micro | Dev |
| Review monitoring alerts and dashboards | Weekly | N/A | Tech Lead / SRE |
| Review and triage customer-reported bugs | Weekly | N/A | PM |
| Apply security patches (OS, runtime, libs) | Bi-weekly | micro or light-spec | Dev |
| Database backup verification | Weekly | N/A | DevOps |

### Periodic (Monthly / Quarterly)

| Activity | Frequency | Spec Mode | Owner |
|----------|-----------|-----------|-------|
| Dependency major version updates | Monthly | light-spec | Dev |
| Performance review (response times, resource usage) | Monthly | N/A | Tech Lead |
| SSL certificate renewal check | Monthly | micro | DevOps |
| Cost review (cloud spend, licensing) | Quarterly | N/A | PM + DevOps |
| Disaster recovery drill | Quarterly | N/A | DevOps |
| Security penetration test | Quarterly or annually | N/A | Security Lead |

### On-Demand

| Activity | Trigger | Spec Mode | Owner |
|----------|---------|-----------|-------|
| Hotfix for production bug | User report / alert | `/aidlc-hotfix` | Dev |
| Feature request from customer | Customer / PM | `/aidlc-intake` (light or feature) | PM → Dev |
| Infrastructure scaling | Load increase / alerts | light-spec or feature-spec | DevOps |
| Compliance / audit response | External request | varies | Tech Lead |

## Maintenance Workflow

### Bug Reports

```
Customer report → PM triages severity
    │
    ├── P0/P1: /aidlc-hotfix (emergency)
    ├── P2: /aidlc-intake → light-spec (next sprint)
    └── P3: Add to backlog (future sprint)
```

### Dependency Updates

```
Security scanner detects vulnerability
    │
    ├── CRITICAL/HIGH: micro or light-spec (immediate)
    │   Fix → Test → Deploy (within 48h)
    │
    └── MEDIUM/LOW: Batch into monthly update cycle
        /aidlc-intake → light-spec → build → review → release
```

### Feature Requests During Maintenance

Even in maintenance mode, small features may be requested:
1. Run `/aidlc-intake` to classify
2. Most maintenance features are light-spec or feature-spec
3. Follow the standard AIDLC flow
4. Keep sprint cycles (can be longer: 3-4 weeks instead of 2)

## Capacity Planning

### Skeleton Crew Model

| Role | Allocation | Responsibilities |
|------|-----------|-----------------|
| Tech Lead | 20-30% | Review, architecture decisions, escalation |
| Developer | 50-80% | Bug fixes, dependency updates, small features |
| DevOps | 10-20% | Infrastructure, monitoring, deployments |
| PM | 10-20% | Customer liaison, triage, backlog management |

### When to Exit Maintenance Mode

Re-enter active development when:
- Major new feature is planned (> feature-spec scope)
- Technology migration is needed
- Significant re-architecture required
- Team scaling up again

Transition: Run `/aidlc-intake` with the new scope → likely full-spec.

## Knowledge Base

During maintenance, accumulate operational knowledge:

```
aidlc-docs/operations/
├── runbooks/              # How to fix common issues
├── faq.md                 # Frequently asked questions
├── known-issues.md        # Known issues and workarounds
└── monitoring/            # Dashboard links and alert definitions
```

### FAQ Template

```markdown
## Q: <common question>
**A**: <answer>
**Related**: <link to runbook or spec>
**Last verified**: YYYY-MM-DD
```

### Known Issues Template

```markdown
## ISSUE-NNN: <title>

- **Status**: Known / Workaround Available / Won't Fix
- **Impact**: Low / Medium / High
- **Workaround**: <description>
- **Root cause**: <if known>
- **Plan**: Fix in Sprint N / Won't fix (reason) / Deferred
```

## Documentation Drift Detection

Over time, documentation can drift out of sync with the actual codebase. Detect and fix drift regularly:

### Periodic Check (Quarterly)

| Check | Method | Owner |
|-------|--------|-------|
| API docs match actual endpoints | Compare `contracts/api-spec.yaml` with route definitions | Dev |
| README instructions still work | Follow setup instructions from scratch | New team member or Dev |
| Architecture docs reflect current state | Compare `design-artifacts/` with actual code structure | Tech Lead |
| AGENTS.md matches project reality | Verify tech stack, roles, and commands are current | Tech Lead |
| Per-folder rules match current stack | Check that `backend/rules/`, `frontend/rules/` reflect actual conventions | Dev |

### How to Fix Drift

```
Drift detected
    │
    ├── Minor (outdated version number, wrong path) → micro fix
    ├── Moderate (missing section, outdated diagram) → light-spec
    └── Major (architecture changed, docs fundamentally wrong) → feature-spec via /aidlc-modify
```

### Prevention

- `/aidlc-release` shrink process removes stale artifacts (reduces surface area for drift)
- PR review checklist includes: "Did you update relevant docs?"
- Sprint retrospective includes: "Are our docs still accurate?"

## Maintenance Sprint Template

Maintenance sprints are lighter than development sprints:

```markdown
# Maintenance Sprint YYYY-MN

## Duration: YYYY-MM-DD → YYYY-MM-DD (3-4 weeks)

## Priority Items
- [ ] Security patches: <list>
- [ ] Customer bugs: <list with severity>

## Routine
- [ ] Dependency audit
- [ ] Backup verification
- [ ] Monitoring review

## Improvements (if capacity allows)
- [ ] <tech debt item>
- [ ] <performance improvement>

## Velocity: N pts (target: N -- lower than dev sprints)
```
