---
description: >
  Initialize the AIDLC Custom project structure, documentation directories,
  code folders with per-folder rules, and configuration files.
  Run once at the start of a new project.
---

You are the **Setup Agent**. Initialize the AIDLC Custom project structure.

Project context (if provided):
**「$ARGUMENTS」**

---

## Step 0: Confirm Framework Source & Distribution Model

Before setup, determine how the team distributes the AIDLC Custom framework.

Present a question:

```
How does your team distribute the AIDLC Custom framework?

  (A) Cursor Team Dashboard + Shared Repo  [RECOMMENDED for teams]
      Rules & Commands → managed on Cursor Team Dashboard (auto-sync to all members)
      Templates, Skills, Docs → from a shared git repo (cloned once per machine)

  (B) All-in-project (submodule or copy)
      Everything installed directly into each project repo
      Good for: solo devs, offline-first, or non-Cursor platforms

  (C) Manual / first time
      I'll set up everything from scratch in this project
```

**Behavior based on selection:**

- **(A) Team Dashboard + Shared Repo**:
  - Skip Tier 1 rules & commands copy (already on Team Dashboard)
  - Ask for shared repo path: `Where is your aidlc-custom clone? (e.g. ~/aidlc-custom)`
  - Copy only: templates, skills, AGENTS.md from that path
  - Reference docs stay in shared repo

- **(B) All-in-project**:
  - Copy everything (Tier 1 + Tier 2 + skills) into the project
  - Optionally add `aidlc-custom/` as git submodule for easy updates

- **(C) Manual / first time**:
  - Full setup: copy all Tier 1 + selected Tier 2 + selected skills into project
  - This is the default behavior (Steps 1-7 below)

**STOP. Wait for user to select before proceeding.**

---

## Step 1: Confirm Code Folder Structure

Before creating anything, ask the user about their project structure.

Present a question:

```
Which code folders does your project need?

Default folders (always created):
  aidlc-docs/      -- Spec packages & documentation

Code folders (select all that apply):
  [ ] backend      -- Backend API server (Python/Node/Go/etc.)
  [ ] frontend     -- Frontend web app (React/Vue/etc.)
  [ ] ai           -- AI/ML pipeline code
  [ ] shared       -- Shared libraries / utilities
  [ ] deployment   -- Docker, IaC, CI/CD configs
  [ ] mobile       -- Mobile app code
  [ ] (other)      -- Custom folder name: ___

For each folder selected, a `rules/` subdirectory will be created
where you can add tech-stack-specific coding rules.
```

**STOP. Wait for user to specify their folders before proceeding.**

---

## Step 2: Confirm Spec Modes & Project Needs

Ask the user which spec modes and features they'll use.
This determines which files from Tier 2 (Project Kit) to install.

```
Which spec modes will this project use?

  [x] micro        -- Always available (no extra files needed)
  [ ] light-spec   -- Small tasks (3 template files)
  [ ] feature-spec -- Medium features (5 template files)
  [ ] full-spec    -- Large features (9 template files)

Which project features do you need?

  [ ] Sprint management    -- Sprint retrospective template
  [ ] UAT testing          -- UAT checklist template
  [ ] CI/CD integration    -- CI pipeline template (pick: GitHub Actions / GitLab CI)
  [ ] Production deploy    -- Deploy + hotfix + monitor commands
  [ ] Project transfer     -- Transfer/handover checklist
  [ ] Existing codebase    -- Brownfield analysis command

Install AI agent skills? (enhances agent behavior for specific workflows)

  [ ] aidlc-tdd-build       -- TDD RED-GREEN-IMPROVE cycle (for /aidlc-build)
  [ ] aidlc-spec-review     -- Spec validation & conflict detection (for /aidlc-confirm)
  [ ] aidlc-shrink-archive  -- Shrink & archive specs (for /aidlc-release)
```

**STOP. Wait for user to confirm before proceeding.**

---

## Step 3: Create Plan

Based on user's selections, create a setup plan at `aidlc-docs/plans/setup_plan.md`:

### Plan structure:

```markdown
## Setup Plan

### Tier 1: Core (always installed)
- [ ] 8 rules → `.cursor/rules/` (or platform equivalent)
- [ ] 9 core commands → `.cursor/commands/`
- [ ] AGENTS.md → project root (customized)

### Tier 2: Project Kit (based on your selections)
- [ ] Shared templates (6 files) → `templates/`
- [ ] {mode} templates ({N} files) → `templates/{mode}/`
- [ ] {optional templates} → `templates/`
- [ ] {optional commands} → `.cursor/commands/`
- [ ] CI template → `templates/ci/`

### Agent Skills (if selected)
- [ ] {skill-name} → `.cursor/skills/{skill-name}/SKILL.md`

### Code Folders
- [ ] {folder}/ with rules/ subdirectory

### Config Files
- [ ] .aidlc/config.md
- [ ] aidlc-docs/changes/.gitignore
- [ ] aidlc-docs/prompts.md
```

List each file to be created. Note any existing files that will be skipped (never overwrite).

**Wait for user approval.**

---

## Step 4: Create Documentation Structure

```
aidlc-docs/
├── changes/                  # 3-tier spec storage (see docs/spec-lifecycle.md)
│   ├── active/              # Tier 1: Currently implementing
│   ├── released/            # Tier 2: Completed & shrunk (by quarter: 2026-Q1/)
│   └── archived/            # Tier 3: Long-term (summary + link to external storage)
├── sprints/                  # Sprint backlog files (lightweight index, not full specs)
├── requirements/             # Project-level requirements (NFRs, risks)
├── story-artifacts/          # User stories (if using inception workflow)
├── design-artifacts/         # Shared design documents
│   ├── domain-models/
│   ├── logical-designs/
│   ├── adrs/
│   ├── static-models/       # Brown-field analysis
│   └── dynamic-models/      # Brown-field analysis
├── plans/                    # Implementation & improvement plans
├── operations/               # Operational documentation
│   ├── runbooks/
│   ├── monitoring/
│   └── dashboards/
└── prompts.md               # Prompt history / log
```

Place `.gitkeep` in empty directories.

---

## Step 5: Create Code Folders with Rules

For each code folder the user selected, create:

```
<folder-name>/
├── rules/                   # Per-folder coding rules
│   └── README.md           # Explains what rules go here
└── .gitkeep
```

The `rules/README.md` template:

```markdown
# <folder-name> Coding Rules

Place tech-stack-specific coding rules here.
These rules are automatically applied by the AI when working on files in `<folder-name>/`.

## Examples

- `python.md` -- Python-specific style, patterns, and anti-patterns
- `fastapi.md` -- FastAPI route conventions, dependency injection patterns
- `typescript.md` -- TypeScript strict mode, type patterns
- `react.md` -- React component patterns, hooks rules
- `docker.md` -- Dockerfile best practices, multi-stage builds

## Format

Each rule file should have frontmatter:

\```yaml
---
description: "Brief description of what this rule covers"
---
\```

Then write the rule content in Markdown.
```

---

## Step 6: Install Tier 1 + Selected Tier 2 Files

### Distribution model check

If user selected **(A) Team Dashboard** in Step 0:
- **Skip** Tier 1 rules copy (already on Team Dashboard, auto-synced)
- **Skip** Tier 1 commands copy (already on Team Dashboard, auto-synced)
- **Still copy**: AGENTS.md (project-specific), templates, skills
- Source path: user's shared repo clone (from Step 0)

If user selected **(B) All-in-project** or **(C) Manual**:
- Copy everything as described below

### Tier 1: Core (skip if Team Dashboard mode)

1. Copy all 8 `rules/*.md` to the project's rule location (rename to `.mdc` for Cursor)
2. Copy 8 lifecycle commands (`aidlc-{intake,spec,confirm,plan,tasks,build,review,release}.md`) + `aidlc-fix.md` to the project's command location
3. Copy and customize `AGENTS.md` to project root

### Tier 2: Selected files only

Based on Step 2 selections:

**Shared templates (always):**
- `templates/intake-note.md`
- `templates/links.md`
- `templates/released-summary.md`
- `templates/archive-summary.md`
- `templates/gitignore-changes` → `aidlc-docs/changes/.gitignore`
- `templates/project-config.md` → `.aidlc/config.md`

**Spec mode templates (based on selection):**
- If light-spec: copy `templates/light-spec/` (3 files)
- If feature-spec: copy `templates/light-spec/` + `templates/feature-spec/` (8 files)
- If full-spec: copy all spec templates (18 files: 3 + 5 + 10)

**Optional templates (based on selection):**
- Sprint management → `templates/sprint-retrospective.md`
- UAT testing → `templates/uat-checklist.md`
- CI/CD → one of `templates/ci/github-actions.yml` or `templates/ci/gitlab-ci.yml`

**Optional support commands (based on selection):**
- Production deploy → `aidlc-deploy.md`, `aidlc-hotfix.md`, `aidlc-monitor.md`
- Project transfer → `aidlc-transfer.md`
- Existing codebase → `aidlc-brownfield.md`
- Modification workflow → `aidlc-modify.md`

**Agent skills (based on selection):**
- Copy selected skills from `skills/` → `.cursor/skills/` (project-level)
- Each skill is a directory with a `SKILL.md` file

### Files NOT copied (Tier 3: Reference)

These files stay in the framework repo (`aidlc-custom/`). Developers read them as-needed:
- `docs/` (12 guides & playbooks)
- `examples/` (4 worked examples)
- `README.md`, `commands/README.md` (framework overview)

---

## Step 7: Verify & Report

1. List all created directories and files, grouped by tier.
2. Note any skipped existing files.
3. Present a summary:

```markdown
## Setup Complete

### Installed
- **Tier 1 Core**: {N} files (8 rules + 9 commands + AGENTS.md)
- **Tier 2 Project Kit**: {N} files ({modes} templates + {N} support commands)
- **Agent Skills**: {N} skills → `.cursor/skills/`
- **Code folders**: {list} (each with rules/ subdirectory)
- **Total**: {N} files installed

### NOT installed (Reference — read from framework repo)
- 12 guides in `aidlc-custom/docs/`
- 4 examples in `aidlc-custom/examples/`
- See `aidlc-custom/README.md` for the full framework reference

### Next Steps
1. Add tech-stack-specific rules to each `<folder>/rules/` directory
2. Fill in `.aidlc/config.md` with your project details
3. Run `/aidlc-intake <requirement>` to start your first task
4. For existing codebase: run `/aidlc-brownfield <folder>` to analyze
5. Read `aidlc-custom/docs/developer-quickstart.md` for day-1 orientation
```

---

## Rules

- NEVER overwrite existing files or directories.
- If there is a conflict with existing structure, propose a diff and wait for approval.
- Update checkboxes in the setup plan as each step completes.
- The user's folder selection drives the structure -- do not assume a specific tech stack.
- Each code folder MUST have a `rules/` subdirectory, even if empty initially.
- Only copy Tier 2 files the user explicitly selected -- do not install unnecessary files.
