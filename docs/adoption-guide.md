# AIDLC Custom -- Adoption Guide

## Understanding the Framework Size

The framework repo contains 75 files, but your project only installs **28-48 files** (Tier 1 + selected Tier 2 + optional skills). See `README.md` → "Installation Tiers" for the full breakdown.

## For New Teams

### Week 1: Foundation

1. **Run `/aidlc-setup`** — it guides you through installation:
   - Asks which code folders you need (backend, frontend, etc.)
   - Asks which spec modes you'll use (start with micro + light-spec)
   - Asks which project features you need (CI, deploy, etc.)
   - Installs only the relevant files (Tier 1 + selected Tier 2)

2. **Start with micro and light-spec** for your first 2-3 tasks. This builds familiarity with minimal overhead.
   - Typo fix, config change → micro (just fix and commit)
   - Bug fix, small feature → light-spec (3 artifacts, 6-step flow)

### Week 2-3: Feature Work

4. **Graduate to feature-spec** for your first real feature. This introduces:
   - Test planning
   - Acceptance criteria
   - Dependency-managed task breakdown

5. **Establish team workflow**:
   - Agree on branch naming convention
   - Set up daily sync cadence
   - Practice spec locking protocol

### Month 2+: Full Capability

6. **Use full-spec** for the first complex, multi-team feature. This adds:
   - Research documents
   - Data model specifications
   - API/event contracts
   - Rollout planning
   - UAT / staging deployment

## Distribution Models

### Model A: Cursor Team Dashboard + Shared Repo (recommended for teams)

```
Cursor Team Dashboard                Shared Git Repo (aidlc-custom/)
══════════════════════               ════════════════════════════════
✅ 8 Rules (auto-sync)               ✅ Templates (copy khi setup)
✅ 16 Commands (auto-sync)            ✅ Skills (copy khi setup)
   → mọi member tự có                ✅ Docs (đọc reference)
   → update 1 lần, sync hết          ✅ Examples (đọc reference)
```

**Setup flow:**

1. **Admin/Tech Lead**: upload rules & commands lên Cursor Team Dashboard (1 lần)
2. **Mỗi member**: clone `aidlc-custom` repo về máy (1 lần)
3. **Mỗi project mới**: chạy `/aidlc-setup` → chọn mode **(A) Team Dashboard**
   - Setup agent hỏi path tới shared repo → copy templates + skills vào project
   - Rules & commands đã có sẵn qua Team Dashboard, không cần copy

**Update flow:**

- Rules/commands thay đổi → Admin update trên Dashboard → auto-sync tất cả members
- Templates/skills thay đổi → `git pull` ở shared repo → chạy lại `/aidlc-setup` hoặc copy thủ công

**Ưu điểm**: Project repo nhẹ, rules luôn up-to-date, không cần mỗi project tự quản rules.

### Model B: All-in-project (submodule hoặc copy)

```
my-project/
├── .cursor/
│   ├── rules/          ← copy từ aidlc-custom/rules/
│   ├── commands/       ← copy từ aidlc-custom/commands/
│   └── skills/         ← copy từ aidlc-custom/skills/
├── templates/          ← copy từ aidlc-custom/templates/
├── AGENTS.md
├── aidlc-docs/         ← sinh ra khi chạy workflow
└── src/
```

**Khi nào dùng**: solo dev, offline-first, non-Cursor platforms (Antigravity, Claude Code), hoặc muốn lock version cụ thể.

**Update flow**: `git submodule update` hoặc copy lại từ framework repo.

### Model C: Manual / First-time

Chạy `/aidlc-setup` chọn mode **(C)** → copy toàn bộ Tier 1 + Tier 2 vào project. Phù hợp cho lần đầu thử nghiệm.

### So sánh

| Khía cạnh | Model A (Dashboard) | Model B (All-in-project) | Model C (Manual) |
|-----------|-------------------|------------------------|-----------------|
| Rules update | Auto-sync | Manual copy/submodule | Manual copy |
| Project repo size | Nhẹ (~templates + skills) | Nặng hơn (~75 files extra) | Nặng |
| Offline access | Cần clone shared repo | Luôn có | Luôn có |
| Multi-platform | Cursor only | Cursor, Antigravity, Claude Code | Tất cả |
| Team sync | Tự động | Manual | Manual |
| Phù hợp | Team >= 2, dùng Cursor | Solo / multi-platform | Pilot / thử nghiệm |

## Platform Setup

### Cursor

After `/aidlc-setup`, your project structure will look like:

```
.cursor/
├── rules/                          # Tier 1: 8 rules (always installed)
│   ├── 01-aidlc-core.mdc          # alwaysApply: true (slim, ~55 lines)
│   ├── 02-spec-modes.mdc          # description-based (loaded when relevant)
│   ├── 03-team-workflow.mdc        # description-based
│   ├── 04-coding-style.mdc        # globs: code files
│   ├── 05-security.mdc            # globs: code files
│   ├── 06-testing.mdc             # globs: test files
│   ├── 07-ai-workflow.mdc         # description-based
│   └── 08-spec-compliance.mdc     # description-based
└── commands/                       # Tier 1 + selected Tier 2
    ├── aidlc-intake.md             # ← 9 core commands (always)
    ├── aidlc-spec.md
    ├── aidlc-confirm.md
    ├── aidlc-plan.md
    ├── aidlc-tasks.md
    ├── aidlc-build.md
    ├── aidlc-review.md
    ├── aidlc-release.md
    ├── aidlc-fix.md
    ├── aidlc-deploy.md             # ← support commands (if selected)
    └── ...

templates/                          # Tier 2: only selected modes
├── intake-note.md                  # shared (always)
├── light-spec/                     # if selected
├── feature-spec/                   # if selected
└── ...

AGENTS.md                           # customized for project
.aidlc/config.md                    # project-specific settings
```

Rename `.md` → `.mdc` for Cursor rules. Keep frontmatter as-is (alwaysApply / globs / description).

### Google Antigravity

```
.agents/
├── rules/
│   └── ...                         # Same files, .md extension
└── workflows/
    └── ...                         # Commands become workflows
```

Frontmatter format:
```yaml
---
description: "Workflow description"
globs: ["**/*.ts"]
---
```

### Claude Code

```
.claude/
├── rules/
│   └── ...
└── commands/
    └── ...
```

Frontmatter uses `paths:` instead of `globs:`.

## SKILLS Lifecycle (Common vs Project)

This framework is the **Common SKILLS** baseline. Each project installs a subset and customizes. Improvements flow back via Sprint Retrospectives and SIP proposals.

See `docs/skills-lifecycle.md` for the full lifecycle, SIP templates, and versioning strategy.

## Tips for Success

1. **Start micro, go heavy as needed** -- resist the urge to use full-spec for everything.
2. **The confirmation gate is sacred** -- never skip `/aidlc-confirm`, even for light-spec.
3. **Spec before code** -- writing the proposal and design first prevents rework.
4. **Templates are guides, not prisons** -- adapt them to your specific change.
5. **Traceability pays off** -- linking artifacts saves time during debugging and audits.
6. **Daily sync prevents conflicts** -- 15 minutes of coordination saves hours of merge conflicts.
7. **Right tool for right job** -- don't apply heavy framework overhead to trivial tasks.
8. **Don't delete knowledge -- shrink it** -- use 3-tier storage (see `docs/spec-lifecycle.md`).
9. **Sprint = index, not storage** -- sprint backlogs link to changes, don't duplicate specs.

## Token Optimization (Reducing AI Context Cost)

The framework is designed to minimize token usage per interaction:

### Rule Loading Strategy

| Rule | Load Strategy | Reason |
|------|--------------|--------|
| `01-aidlc-core.md` (~55 lines) | **alwaysApply** | Core principles, always needed |
| `02-spec-modes.md` | **description-based** (manual) | Only needed when selecting modes |
| `03-team-workflow.md` | **description-based** (manual) | Only needed for team/conflict topics |
| `04-coding-style.md` | **globs** (code files) | Only when editing code |
| `05-security.md` | **globs** (code files) | Only when editing code |
| `06-testing.md` | **globs** (test files) | Only when editing tests |
| `07-ai-workflow.md` | **description-based** (manual) | Rarely needed |
| `08-spec-compliance.md` | **description-based** (manual) | Only when changing specs/contracts |

### How It Works

- **alwaysApply**: Loaded into every interaction (keep these small!)
- **globs**: Loaded only when editing files matching the glob pattern
- **description-based**: AI reads the `description` field and decides whether to load the full file

### Best Practices

- Keep `alwaysApply` rules under 100 lines total
- Each command file already defines its agent role -- don't duplicate in AGENTS.md
- AGENTS.md is a router (~90 lines), not an encyclopedia
- Commands load spec artifacts selectively (only what the current step needs)
- Write clear `description` fields so AI can make good loading decisions

### Estimated Savings

Compared to loading all rules on every interaction:
- **Before optimization**: ~2,000 lines loaded per interaction
- **After optimization**: ~100-200 lines (alwaysApply only) + relevant globs
- **Savings**: ~60-70% fewer tokens per interaction

## Command Quick Reference

See `commands/README.md` for the full command catalog and flow diagrams per spec mode.
