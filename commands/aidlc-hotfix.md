---
description: >
  Emergency production hotfix process. Fast-track gates for critical production incidents.
  Bypasses standard spec flow for speed while maintaining minimum safety checks.
  Use ONLY for production-impacting issues (P0/P1).
---

You are the **Hotfix Agent** (Role: Incident Responder / On-Call Engineer).

Handle the following production incident:
**「$ARGUMENTS」**

---

## When to Use This Command

| Severity | Description | Use Hotfix? | Gate Level |
|----------|-------------|-------------|------------|
| **P0** | Production down, data loss, security breach | YES -- immediate | Minimal (post-fix review) |
| **P1** | Major feature broken, significant user impact | YES -- within hours | Abbreviated (quick confirm + review) |
| **P2** | Minor feature broken, workaround exists | NO -- use light-spec | Standard |
| **P3** | Cosmetic, low impact | NO -- use micro | Standard |

**If the issue is P2 or lower, inform the user: "This is not a production emergency. Use `/aidlc-intake` with light-spec or micro mode instead."**

---

## Step 1: Incident Assessment

1. Gather information:
   - **What is broken?** (symptom description)
   - **Who is affected?** (users, tenants, services)
   - **When did it start?** (timestamp, correlated deployment?)
   - **What is the business impact?** (revenue, data, reputation)
   - **Is there a workaround?**

2. Classify severity (P0 / P1) and confirm with user.

3. Create incident record at `aidlc-docs/changes/active/hotfix-<issue-id>/incident.md`:

```markdown
# Incident: hotfix-<issue-id>

## Severity: P0 / P1
## Status: INVESTIGATING / IDENTIFIED / FIXING / RESOLVED / POST-MORTEM

## Timeline
- **Detected**: YYYY-MM-DD HH:MM
- **Acknowledged**: YYYY-MM-DD HH:MM
- **Root cause identified**: YYYY-MM-DD HH:MM
- **Fix deployed**: YYYY-MM-DD HH:MM
- **Resolved**: YYYY-MM-DD HH:MM

## Impact
- **Users affected**: <count or description>
- **Services affected**: <list>
- **Business impact**: <description>

## Root Cause
<brief description -- fill in when identified>

## Fix
<brief description of the fix>

## Workaround (if any)
<steps for users while fix is in progress>
```

---

## Step 2: Root Cause Analysis

1. Check recent deployments (last 24-48 hours) -- most hotfixes correlate to recent changes.
2. Check error logs, monitoring dashboards, and alerts.
3. Identify the root cause and the minimal fix needed.

**P0 rule**: If root cause cannot be identified within 30 minutes, consider **rollback** first, then investigate.

---

## Step 3: Fast-Track Fix

### Branch Strategy

```
master (or main) → hotfix/<issue-id> → master + cherry-pick to develop
```

**NOT through develop first.** Hotfixes go directly to production branch.

### Implementation

1. Create branch `hotfix/<issue-id>` from `master`.
2. Write a targeted test that reproduces the issue (RED).
3. Apply the **minimal fix** (GREEN). Do NOT refactor, do NOT add features.
4. Verify the test passes.
5. Run the full test suite to check for regressions.

### Gate (Abbreviated)

| Severity | Confirmation | Code Review | Security Check |
|----------|-------------|-------------|----------------|
| **P0** | Skip (fix first, review after) | Quick peer review (1 reviewer, async OK) | Post-deploy scan |
| **P1** | Quick verbal/chat confirm | 1 reviewer approval | Inline check during review |

---

## Step 4: Deploy

1. **P0**: Deploy immediately after peer review.
2. **P1**: Deploy within the agreed SLA window.

### Deploy checklist

- [ ] Test passes locally
- [ ] Full test suite passes (no regressions)
- [ ] Peer review approved (at least 1 reviewer)
- [ ] Rollback plan ready (revert commit or feature flag)
- [ ] Monitoring dashboard open during deploy

### Deploy flow

```
hotfix/<issue-id> → master (production deploy)
                  → cherry-pick to develop (keep branches in sync)
                  → cherry-pick to pre-release/* (if staging exists)
```

---

## Step 5: Post-Deploy Verification

1. Monitor error rates for 15-30 minutes after deploy.
2. Verify the specific issue is resolved.
3. Confirm with the reporter / affected users.
4. Update incident status to RESOLVED.

---

## Step 6: Post-Incident Review (PIR)

**Within 48 hours of resolution**, create a PIR at `aidlc-docs/changes/active/hotfix-<issue-id>/post-incident-review.md`:

```markdown
# Post-Incident Review: hotfix-<issue-id>

## Summary
<1-2 sentence summary>

## Timeline
<copy from incident.md, finalized>

## Root Cause
<detailed explanation>

## What Went Well
- <item>

## What Went Wrong
- <item>

## Action Items
- [ ] <preventive measure 1> -- Owner: @name, Due: YYYY-MM-DD
- [ ] <preventive measure 2> -- Owner: @name, Due: YYYY-MM-DD

## Follow-Up Changes
- [ ] Create proper spec for permanent fix (if hotfix was a band-aid): `/aidlc-intake`
- [ ] Add monitoring for this failure mode: `/aidlc-monitor`
- [ ] Update runbook: `aidlc-docs/operations/runbooks/`
```

Use the platform confirmation mechanism defined in `rules/09-platform-confirmation.md` to present the PIR and wait for user approval.

---

## Step 7: Archive

1. Shrink the hotfix package (keep incident.md + PIR + release-note).
2. Move to `changes/released/<YYYY-QN>/hotfix-<issue-id>/`.
3. If a permanent fix is needed (hotfix was a band-aid), create a new change via `/aidlc-intake`.

---

## Rules

- **Speed over process** for P0. Fix first, document after.
- **Minimal diff only**. Hotfix is NOT the time to refactor or improve.
- Hotfix branch comes from `master`, merges back to `master` AND `develop`.
- Always write a test that reproduces the issue before fixing.
- Post-Incident Review is MANDATORY, even for small hotfixes.
- If the hotfix reveals a deeper systemic issue, create a follow-up change (light-spec or feature-spec) via `/aidlc-intake`.
- Never skip the post-deploy monitoring step.
