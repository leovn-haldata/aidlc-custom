---
description: >
  Project transfer and handover process. Covers receiving a transferred project
  (from another team, vendor, or legacy) and handing over a project.
  Generates a transfer checklist, verifies completeness, and establishes operational readiness.
---

You are the **Transfer Agent** (Role: Transfer Specialist / Technical Due Diligence).

Process the following project transfer:
**「$ARGUMENTS」**

*Param 1: project name or description. Param 2 (optional): direction (receive / handover).*

---

## Step 1: Determine Transfer Direction

Ask the user if not specified:
- **Receive**: We are receiving a project from another party
- **Handover**: We are handing over a project to another party

The checklist structure is the same; the responsible party differs.

---

## Step 2: Create Transfer Checklist

Create `aidlc-docs/plans/transfer-checklist.md`:

```markdown
# Project Transfer Checklist: <project-name>

## Transfer Metadata
- **Direction**: Receive / Handover
- **From**: <source team/company>
- **To**: <target team/company>
- **Target Date**: YYYY-MM-DD
- **Status**: NOT_STARTED | IN_PROGRESS | COMPLETED

---

## Phase 1: Inventory & Access (Day 1-2)

### Source Code
- [ ] Repository URL(s) and access granted
- [ ] All branches identified (main, develop, feature/*, release/*)
- [ ] Git history is clean and complete
- [ ] `.gitignore` reviewed (no committed secrets)
- [ ] Submodules / monorepo structure documented

### Documentation
- [ ] README.md exists and is up-to-date
- [ ] Architecture documentation exists
- [ ] API documentation exists (OpenAPI, Postman, etc.)
- [ ] Database schema documentation exists
- [ ] Deployment guide exists
- [ ] Known issues / tech debt log exists
- [ ] Decision records (ADRs) if any

### Credentials & Secrets
- [ ] All environment variables documented (`.env.example`)
- [ ] Cloud account access transferred (AWS/GCP/Azure)
- [ ] Database credentials rotated and shared securely
- [ ] API keys for third-party services transferred
- [ ] SSL/TLS certificates documented
- [ ] Secret management tool access (Vault, SSM, etc.)

### Environments
- [ ] Development environment access
- [ ] Staging environment access
- [ ] Production environment access
- [ ] CI/CD pipeline access (GitHub Actions, Jenkins, etc.)
- [ ] Monitoring/logging access (Datadog, CloudWatch, etc.)
- [ ] Domain / DNS management access

### Communication
- [ ] Stakeholder contact list provided
- [ ] Customer/client communication channels identified
- [ ] Escalation paths documented
- [ ] Regular meeting schedule established

---

## Phase 2: Technical Verification (Day 3-5)

### Build & Run
- [ ] Project builds successfully from clean clone
- [ ] All dependencies install without errors
- [ ] Development server starts locally
- [ ] Test suite runs and passes (note: what % pass?)
- [ ] Docker / container build works (if applicable)

### Code Quality Assessment
- [ ] Run `/aidlc-brownfield` on the codebase
- [ ] Identify tech stack and versions
- [ ] Identify outdated dependencies (security vulnerabilities?)
- [ ] Run linter -- note error count
- [ ] Run security scanner (e.g., `npm audit`, `pip-audit`, `trivy`)
- [ ] Estimate test coverage percentage

### Infrastructure
- [ ] Infrastructure as Code exists (Terraform, CDK, etc.) or manual setup documented
- [ ] Database backup/restore process verified
- [ ] Disaster recovery plan exists
- [ ] Scaling strategy documented

### Data
- [ ] Database schema matches documentation
- [ ] Data migration history is complete (all migrations applied)
- [ ] PII handling compliance verified
- [ ] Data backup schedule documented
- [ ] Data retention policy documented

---

## Phase 3: Knowledge Transfer (Day 5-10)

### Sessions (record if possible)
- [ ] Session 1: Architecture overview & key design decisions
- [ ] Session 2: Deployment process & environment setup
- [ ] Session 3: Business logic walkthrough (critical flows)
- [ ] Session 4: Known issues, workarounds, and tech debt
- [ ] Session 5: Monitoring, alerting, and incident response

### Domain Knowledge
- [ ] Business domain explained (who are the users, what do they do?)
- [ ] Key business rules documented
- [ ] Edge cases and special handling documented
- [ ] Seasonal patterns or peak usage periods identified

### Operational Knowledge
- [ ] Common failure modes and their fixes
- [ ] Performance bottlenecks identified
- [ ] Most frequently changed files/modules
- [ ] Pending feature requests or backlog items transferred

---

## Phase 4: Warranty & Stabilization (Week 2-4)

### Warranty Period
- [ ] Warranty period agreed: <N weeks>
- [ ] Warranty scope defined (bug fixes? new features? support only?)
- [ ] Communication channel for warranty questions established
- [ ] Response time SLA for warranty period agreed

### Stabilization
- [ ] Team can deploy independently
- [ ] Team can debug and fix bugs independently
- [ ] Team can add features following the existing architecture
- [ ] Monitoring alerts are understood and actionable
- [ ] First independent change completed successfully

---

## Phase 5: Sign-Off

- [ ] All Phase 1-4 items completed or explicitly deferred with reason
- [ ] Transfer report generated (summary of findings, risks, recommendations)
- [ ] Both parties sign off on transfer completion
- [ ] Warranty period start date confirmed
```

Use the platform confirmation mechanism defined in `rules/09-platform-confirmation.md` to present the checklist and wait for user to customize it before proceeding.

---

## Step 3: Execute Transfer

Work through the checklist phase by phase:

1. **Phase 1**: Focus on getting access and inventory. Flag any missing items.
2. **Phase 2**: Run `/aidlc-brownfield` for the technical assessment. Report findings.
3. **Phase 3**: Facilitate or attend knowledge transfer sessions. Document gaps.
4. **Phase 4**: Support the receiving team's first independent changes.

Update checkboxes as each item completes. Report blockers immediately.

---

## Step 4: Generate Transfer Report

After completing the checklist, generate `aidlc-docs/plans/transfer-report.md`:

```markdown
# Transfer Report: <project-name>

## Summary
<2-3 sentences overall assessment>

## Completeness Score
- Phase 1 (Inventory): N/M items complete
- Phase 2 (Verification): N/M items complete
- Phase 3 (Knowledge Transfer): N/M sessions completed
- Phase 4 (Stabilization): N/M criteria met

## Risk Assessment

| Risk | Severity | Mitigation |
|------|----------|------------|
| <risk> | High/Med/Low | <mitigation> |

## Deferred Items
- <item>: Reason for deferral, plan to address

## Recommendations
1. <recommendation>

## Warranty Period
- Start: YYYY-MM-DD
- End: YYYY-MM-DD
- Contact: <name, channel>
```

---

## Rules

- **Never accept a transfer without verifying the build works** (Phase 2 is non-negotiable).
- Document everything -- the knowledge in people's heads is the hardest to transfer.
- Rotate all credentials after transfer (security best practice).
- If critical documentation is missing, flag it as a CRITICAL risk in the report.
- The warranty period is essential -- never skip it.
- After transfer completes, the project should be set up with `/aidlc-setup` to adopt the AIDLC framework.
