# Example: full-spec -- Multi-Tenant AI Pipeline

## Scenario

Build a new AI processing pipeline that handles citizen survey responses for multiple prefectures. Requires: PII masking (GiNZA), sentiment analysis (Qwen LLM), topic classification, new DB tables, GCS integration, and tenant isolation. Multiple teams involved (backend, AI, infrastructure). Estimated: 3 weeks.

## Flow Used

```
/aidlc-intake "Build multi-tenant AI survey analysis pipeline for Japanese prefectures"
  → Recommended: full-spec (multi-team, DB migration, external integrations, > 2 sprints)
  → User confirms

/aidlc-spec add-ai-pipeline
  → Generated: proposal.md, design.md, tasks.md, test-plan.md, acceptance.md,
               research.md, data-model.md, contracts/api-spec.yaml, contracts/event-spec.md,
               rollout-plan.md

/aidlc-confirm add-ai-pipeline
  → AI pre-review: 2 WARNINGs (PII handling needs explicit masking step, LLM timeout not specified)
  → Human: REVISE (address warnings)
  → Re-spec and re-confirm: CONFIRM

/aidlc-plan add-ai-pipeline
  → Domain model: PipelineRun, ProcessingStep, SurveyResponse entities
  → Architecture: Queue-based pipeline, step-by-step processing, GCS for intermediate results
  → ADR-001: Queue vs direct processing (Queue chosen for reliability)
  → ADR-002: GiNZA local vs cloud NER (Local chosen for PII compliance)
  → Consistency check: All contracts match data model

/aidlc-tasks add-ai-pipeline
  → 4 phases (Data Layer → API → AI Pipeline → Integration)
  → 15 tasks across 2 teams
  → Mapped to Sprint 3 in the project plan

/aidlc-build add-ai-pipeline
  → Sprint 3 execution: DB migration → models → services → API → pipeline steps → tests
  → Coverage: 87% overall, 92% on PII handling

/aidlc-review add-ai-pipeline
  → TDD: PASS (87% coverage)
  → Code Review: 2 WARNINGs, 3 SUGGESTIONs (addressed)
  → Security: 1 WARNING (add rate limiting to pipeline trigger endpoint -- fixed)
  → Tech Lead decision: Accept → merge to develop
  → Deploy to Dev Env

--- Deploy to Staging ---
  → Merge develop → pre-release/v1.0
  → Deploy to Staging

--- UAT ---
  → JP PM/BrSE + Tech Lead validate against spec & customer needs
  → Bug found: PII masking misses phone numbers in free text
  → Fix Bug Loop: Create PR fix (micro mode), merge, re-test → PASS

--- Production ---

/aidlc-release add-ai-pipeline
  → Backup master
  → Rollout Phase 1: Canary with 1 prefecture (3 days)
  → Rollout Phase 2: Full rollout to all prefectures
  → Monitoring: Error rate, processing time, PII detection accuracy dashboards
```

## Artifacts Produced (Active -- 15 files)

```
aidlc-docs/changes/active/add-ai-pipeline/
├── intake-note.md
├── proposal.md
├── design.md
├── tasks.md
├── test-plan.md
├── acceptance.md
├── research.md
├── data-model.md
├── contracts/
│   ├── api-spec.yaml
│   └── event-spec.md
├── rollout-plan.md
├── plan.md
├── build-summary.md
├── review-report.md
└── release-note.md
```

## After Release (Shrunk → Released Tier -- 7 files)

```
aidlc-docs/changes/released/2026-Q2/add-ai-pipeline/
├── README.md          # Summary entry point
├── proposal.md        # Final business case
├── design.md          # Architecture decisions + ADRs
├── data-model.md      # DB schema (kept: breaking change)
├── contracts/         # API contracts (kept: breaking change)
│   ├── api-spec.yaml
│   └── event-spec.md
├── release-note.md
└── links.md           # PR, test results, deploy links
```

Process artifacts (build-plan, review-report, tasks, etc.) are removed during shrink.
For full history, see external archive when Tier 3 is activated.

## Branch Strategy
```
feature/add-ai-pipeline → develop (Dev Env)
develop → pre-release/v1.0 (Staging)
pre-release/v1.0 → master (Production)
```

## Time: ~3 weeks total (across 2 teams)
