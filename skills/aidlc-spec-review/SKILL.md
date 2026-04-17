---
name: aidlc-spec-review
description: >-
  Review a spec package (proposal, design, tasks) for completeness, feasibility,
  and cross-spec conflicts. Use when running /aidlc-confirm, reviewing spec artifacts
  before implementation, or when the user asks to validate a spec.
---

# AIDLC Spec Review

## When to Use

- User runs `/aidlc-confirm` or asks to review/validate a spec
- Before starting implementation on a change
- When checking if a spec conflicts with other active specs

## Review Checklist by Mode

### light-spec (5 items)

1. **Business justification**: Is the "why" clear?
2. **Scope clarity**: Are scope and non-scope defined? Any ambiguity?
3. **Impact assessment**: Are impacted files/components listed?
4. **Risk identification**: Are risks listed with mitigations?
5. **Backward compatibility**: Will this break existing behavior?

### feature-spec (add 5 more)

6. **Architecture fit**: Does the design align with existing architecture?
7. **Test coverage plan**: Is the test plan adequate?
8. **Acceptance criteria**: Are they specific, measurable, testable?
9. **Performance impact**: Could this degrade performance?
10. **Security impact**: New attack surface?

### full-spec (add 5 more)

11. **Data model consistency**: Schema matches design?
12. **Contract completeness**: API/event contracts complete and versioned?
13. **Cross-team dependencies**: Identified and acknowledged?
14. **Rollout plan feasibility**: Realistic? Rollback tested?
15. **Monitoring coverage**: Right metrics, logs, alerts defined?

## Cross-Spec Conflict Detection

When multiple changes are active in the same sprint:

```
1. List all active specs: ls aidlc-docs/changes/active/
2. For each pair of active specs:
   - Do they modify the same files? → File conflict
   - Do they touch the same module/API/DB table? → Logic conflict risk
   - Do they have contradicting design decisions? → Spec conflict
3. Report conflicts with severity:
   - CRITICAL: Same file, contradicting changes
   - WARNING: Same module, may affect each other
   - INFO: Different modules, unlikely conflict
```

## Output Format

```markdown
# Confirmation Report: <change-id>

## Mode: <light-spec / feature-spec / full-spec>

## Summary
<2-3 sentence assessment>

## Findings

### CRITICAL (must fix before proceeding)
- [ ] <finding>

### WARNING (should address, not blocking)
- [ ] <finding>

### SUGGESTION (nice to have)
- [ ] <finding>

## Cross-Spec Conflicts
<conflicts with other active changes, or "None detected">

## Decision: [CONFIRM / REVISE / REJECT]
```

## Rules

- Always present the report and **STOP**. Wait for human decision.
- Do not auto-confirm even if zero issues found.
- If CRITICAL issues exist → recommend REVISE.
- For full-spec: if cross-team dependencies are unsigned → recommend REVISE.
