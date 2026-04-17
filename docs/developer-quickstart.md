# Developer Quickstart (Day-1 Guide)

## What is AIDLC?

An AI-driven development lifecycle where:
- **AI generates** specs, plans, reviews, and code
- **Humans approve** at defined gates (confirm, review, release)
- **4 spec modes** adapt overhead to task complexity

```
micro        → trivial fix, no spec, just commit
light-spec   → small task, 3 artifacts, 6 steps
feature-spec → medium feature, 5 artifacts, 8 steps
full-spec    → large feature, 8+ artifacts, 9+ steps
```

## Cheat Sheet: Which Command When?

### "I need to fix a bug"

| Bug Type | Mode | What to Do |
|----------|------|-----------|
| Typo, 1-line fix | micro | Fix → test → commit. Done. |
| Complex bug (needs tracing) | light-spec | `/aidlc-intake` → `/aidlc-spec` → `/aidlc-confirm` → `/aidlc-build` → `/aidlc-review` → `/aidlc-release` |
| Production is down | hotfix | `/aidlc-hotfix` (fast-track, branch from master) |

### "I need to build a feature"

| Feature Size | Mode | What to Do |
|-------------|------|-----------|
| Small (< 3 days) | light-spec | Same as complex bug above |
| Medium (3-10 days) | feature-spec | Add `/aidlc-plan` + `/aidlc-tasks` between confirm and build |
| Large (> 10 days) | full-spec | Full pipeline including deploy + UAT |

### "I need to do something else"

| Task | Command |
|------|---------|
| Understand an existing codebase | `/aidlc-brownfield` |
| Build is broken | `/aidlc-fix` |
| Change existing feature | `/aidlc-modify` → then follow the flow |
| Set up monitoring | `/aidlc-monitor` |
| Deploy to staging/prod | `/aidlc-deploy` |

## Example Walkthrough: Fix a Complex Bug

```
1. You receive a bug report: "Login times out on slow connections"

2. Run /aidlc-intake with the bug description
   → AI classifies as light-spec, creates intake-note.md
   → You confirm the mode

3. Run /aidlc-spec
   → AI generates: proposal.md, design.md, tasks.md
   → You review

4. Run /aidlc-confirm
   → AI pre-reviews the spec, presents checklist
   → Tech Lead confirms

5. Run /aidlc-build
   → AI creates build-plan.md
   → You approve, then implement with TDD:
     - Write test first (RED)
     - Write minimal fix (GREEN)
     - Refactor (IMPROVE)

6. Run /aidlc-review
   → AI runs 3 parallel reviews: TDD + Code + Security
   → Tech Lead approves PR

7. Run /aidlc-release
   → AI archives spec, generates release note
   → Done!
```

## Daily Workflow

```
Morning:
  1. Pull latest changes
  2. Check tasks.md -- find your [IN_PROGRESS] or pick an [UNASSIGNED] task
  3. Daily sync (15 min): report status, blockers, file overlaps

During work:
  1. Follow TDD: test → implement → refactor
  2. Commit frequently (at least daily)
  3. If blocked → mark task [BLOCKED] with reason

End of day:
  1. Push your changes
  2. Update task status in tasks.md
```

## Key Rules to Remember

1. **Never skip the confirmation gate** (`/aidlc-confirm`) -- even for light-spec
2. **Test first, code second** -- TDD is mandatory
3. **Don't touch someone else's task** -- check `@name [IN_PROGRESS]` markers
4. **Commit per gate** -- makes rollback easy (`git reset`)
5. **No new features after Sprint Planning** -- bug fixes only
6. **Ask if unsure** -- every gate is a chance to ask questions

## Where to Find Things

| What | Where |
|------|-------|
| Project config | `.aidlc/config.md` |
| Current sprint | `aidlc-docs/sprints/sprint-YYYY-SNN.md` |
| Your current change | `aidlc-docs/changes/active/<change-id>/` |
| Coding rules | Per-folder: `backend/rules/`, `frontend/rules/`, etc. |
| Framework rules | `.cursor/rules/` (or equivalent for your platform) |
| Past changes | `aidlc-docs/changes/released/<YYYY-QN>/` |

> **Note**: Not all framework files are in your project. Reference docs and examples stay in the framework repo (`aidlc-custom/`). See `aidlc-custom/README.md` → "Installation Tiers" for what's installed vs. reference-only.

## Further Reading

These documents are in the **framework repo** (`aidlc-custom/docs/`), not your project directory. Read them as-needed:

| Topic | Document |
|-------|----------|
| Spec modes in detail | `rules/02-spec-modes.md` (installed in your project) |
| Team workflow & conflicts | `aidlc-custom/docs/team-conflict-playbook.md` |
| Sprint management | `aidlc-custom/docs/sprint-management.md` |
| Estimation | `aidlc-custom/docs/estimation-guide.md` |
| Full adoption guide | `aidlc-custom/docs/adoption-guide.md` |
| CI/CD mapping | `aidlc-custom/docs/cicd-integration.md` |
| Versioning & changelog | `aidlc-custom/docs/versioning-guide.md` |
