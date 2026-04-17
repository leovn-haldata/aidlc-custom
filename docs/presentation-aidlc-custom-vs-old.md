---
title: "AIDLC Custom -- Framework thay thế hoàn toàn AIDLC Old"
author: "AI-DLC Team"
date: "2026-04-17"
marp: true
theme: default
paginate: true
---

<!-- 
  Slide deck tương thích Marp, Slidev, hoặc đọc trực tiếp trên GitHub/VS Code.
  Mỗi `---` là ranh giới slide.
  Thời lượng khuyến nghị: 30–45 phút.
-->

# AIDLC Custom

## Framework thay thế hoàn toàn AIDLC Old

> Kết hợp **AI-DLC** (operating model) + **OpenSpec** (lightweight execution) + **Spec Kit** (structured execution) thành **4 adaptive spec modes**.

---

# Agenda

1. Bối cảnh & Tại sao cần thay đổi
2. Kiến trúc 2 tầng
3. 4 Spec Modes -- Cải tiến cốt lõi
4. So sánh Command Pipeline
5. Vấn đề #1: Repo Bloat → 3-Tier Storage
6. Vấn đề #2: Token Overhead → Smart Rule Loading
7. Vấn đề #3: Team Collaboration → 3-Lane Workflow
8. Vấn đề #4: Cài đặt phức tạp → Tiered Installation
9. Vấn đề #5: Lifecycle Gaps → Hotfix, UAT, Transfer...
10. Vấn đề #6: Artifact Organization → Per-Change Folders
11. So sánh tổng hợp
12. Lộ trình triển khai
13. Kết luận

---

# Bối cảnh

## AIDLC Old là gì?

- Bản **Nhật hoá** của AI-DLC gốc (AWS Labs), implement bởi **tamagoya** (ai-dlc-sdd)
- Thiết kế **1-size-fits-all**: mọi task đều đi qua flow nặng (9 bước)
- 16 commands, 4 rules `.mdc` (tất cả `alwaysApply`)
- Chỉ hỗ trợ **Cursor**, tài liệu tiếng Nhật

## Tại sao cần thay đổi?

| Vấn đề | Hậu quả |
|--------|---------|
| 1 flow cố định | Fix typo cũng phải qua inception → architecture |
| Không có spec modes | Overhead 60-80% cho task nhỏ |
| Không có team workflow | Xung đột spec, file, logic không được phát hiện sớm |
| Repo phình to | `aidlc-docs/plans/` sinh hàng chục file không dọn dẹp |
| Token lãng phí | ~2000 dòng rules load mỗi tương tác |

---

# AIDLC Custom -- Nguồn gốc

Kết hợp **3 framework** thành 1 hệ thống thống nhất:

| Nguồn | Đóng góp |
|-------|----------|
| **AI-DLC** (AWS Labs) | Operating model, plan-first, human-in-the-loop |
| **OpenSpec** (Fission-AI) | Lightweight spec execution (light-spec mode) |
| **Spec Kit** (GitHub) | Structured spec execution (full-spec mode) |
| **ai-dlc-sdd** (tamagoya) | Reference implementation, bài học thực tế |

> Không phải "phiên bản nâng cấp" -- mà là **framework thay thế hoàn toàn**.

---

# Kiến trúc 2 tầng

```
┌──────────────────────────────────────────────────────────┐
│              Layer 1: AI-DLC Operating Model              │
│   Governance · Quality Gates · Human-in-the-Loop         │
│   Lifecycle · Context Memory · Traceability              │
├──────────────────────────────────────────────────────────┤
│              Layer 2: Spec Execution Layer                │
│                                                          │
│  ┌────────┐  ┌───────────┐  ┌─────────────┐  ┌───────┐  │
│  │ micro  │  │light-spec │  │feature-spec │  │full-  │  │
│  │ no-spec│  │ OpenSpec   │  │  Hybrid     │  │spec   │  │
│  │        │  │  style    │  │             │  │SpecKit│  │
│  └────────┘  └───────────┘  └─────────────┘  └───────┘  │
└──────────────────────────────────────────────────────────┘
```

**Điểm then chốt**: Tách **governance** (không đổi) khỏi **execution** (thay đổi theo độ phức tạp).

- Layer 1 đảm bảo chất lượng **luôn nhất quán**
- Layer 2 cho phép **linh hoạt** theo quy mô task

---

# 4 Spec Modes -- Cải tiến cốt lõi

| Mode | Khi nào dùng | Artifacts | Steps | Thời gian |
|------|-------------|-----------|-------|-----------|
| **micro** | Typo, config, fix 1 dòng | 0 (commit msg) | 2 | < 1h |
| **light-spec** | Bug fix, feature nhỏ < 3 ngày | 3 files | 6 | 1h–3 ngày |
| **feature-spec** | Feature trung bình (3–10 ngày, 1 team) | 5 files | 8 | 3–10 ngày |
| **full-spec** | Feature lớn (> 10 ngày, multi-team) | 8+ files | 9+ | 10+ ngày |

### So sánh trực quan

```
AIDLC Old:    ████████████████████  (9 bước cho MỌI task)
              ↑ fix typo cũng phải qua đây

AIDLC Custom: ██                    micro    (2 bước)
              ██████                light    (6 bước)  
              ████████              feature  (8 bước)
              ████████████████████  full     (9+ bước)
              ↑ đúng overhead cho đúng task
```

> **Giảm 60–80% overhead** cho task nhỏ và trung bình.

---

# So sánh Command Pipeline

## AIDLC Old (16 commands, 1 flow cố định)

```
setup → inception → units → domain-model → architecture
  → code-generation → iac-apis → deployment → monitoring
  + code-review, security-review, test-coverage, build-fix
  + modification, refactor
```

## AIDLC Custom (17 commands, 4 adaptive flows)

```
intake → spec → confirm → [plan → tasks] → build → review → release
  + deploy, monitor, hotfix, transfer, modify, fix, brownfield, setup
```

---

# So sánh Command Pipeline (tiếp)

| Điểm so sánh | Old | Custom |
|--------------|-----|--------|
| Flow cố định | 1 (nặng) | 4 (micro / light / feature / full) |
| Confirmation gate | Không rõ ràng | `/aidlc-confirm` **bắt buộc** |
| Review | 3 command riêng lẻ | 1 `/aidlc-review` (3 expert **song song**) |
| Release/archive | Không có | `/aidlc-release` + 3-tier storage |
| Hotfix | Không có | `/aidlc-hotfix` (P0/P1 fast-track) |
| Transfer/Handover | Không có | `/aidlc-transfer` checklist |
| Task breakdown | Không rõ ràng | `/aidlc-tasks` với dependency graph |

### Flow theo mode

```
micro:        fix → commit
light-spec:   intake → spec → confirm → build → review → release
feature-spec: intake → spec → confirm → plan → tasks → build → review → release
full-spec:    intake → spec → confirm → plan → tasks → build → review → release → deploy
```

---

# Vấn đề #1: Repo Bloat

## Thực trạng với AIDLC Old

Dự án Shiga (trong `old/aidlc-docs/`) sau vài tháng:

- **11 ADRs**, 5 units, 5 domain models, 5 logical designs
- Hàng chục plan files: `modification_analysis_*.md`, `code_generation_*_plan.md`...
- File trùng lặp: `code_generation_all-units_plan.md` vs `code_generation_all_units_plan.md`
- **Không có cơ chế dọn dẹp** → repo phình to vô hạn
- Plans thư mục trở thành "bãi rác" artifacts

## Hậu quả

- `git clone` ngày càng chậm
- AI context window bị lãng phí khi đọc artifacts cũ
- Khó tìm kiếm thông tin liên quan

---

# Giải pháp: 3-Tier Storage

```
changes/active/      →  (shrink)  →  changes/released/     →  (archive)  →  changes/archived/
  Full artifacts          Giữ what+why     Read-only, by quarter      Summary + S3/GCS link
```

### Shrink Rules

| **Giữ lại** | **Xoá / chuyển external** |
|-------------|--------------------------|
| README.md (summary) | build-plan.md |
| proposal.md (final) | build-summary.md, spec-plan.md |
| design.md (nếu chứa ADRs) | review-report.md, deploy-report.md |
| release-note.md | notes.md, debug.md |
| links.md (PR, test, issue) | ai-drafts/, raw AI output |
| data-model.md (nếu DB change) | tasks.md (completed tasks) |

> **Nguyên tắc**: Giữ **what** và **why**. Xoá **how** (code là truth).

### Sprint = Index, không phải Storage

```markdown
# Sprint 2026-S07
- [ ] [CHG-015](../changes/active/CHG-015-user-profile/) @dev-a -- feature-spec
- [ ] [CHG-016](../changes/active/CHG-016-fix-timeout/) @dev-b -- light-spec
```

---

# Vấn đề #2: Token / Context Window Overhead

## Thực trạng với AIDLC Old

- **4 rules** `.mdc` tất cả `alwaysApply: true`
- `AGENTS.md` dài và chi tiết (role definitions lặp lại nội dung commands)
- **~2,000 dòng** load vào mỗi tương tác, bất kể task là gì
- Fix typo cũng bị inject đầy security guidelines + testing rules

---

# Giải pháp: Smart Rule Loading

| Rule | Chiến lược load | Khi nào |
|------|-----------------|---------|
| `01-aidlc-core` (~55 dòng) | **alwaysApply** | Mọi tương tác (nhẹ!) |
| `02-spec-modes` | **description-based** | Chỉ khi chọn mode |
| `03-team-workflow` | **description-based** | Chỉ khi cần team protocol |
| `04-coding-style` | **globs** (`*.ts, *.js...`) | Chỉ khi edit code |
| `05-security` | **globs** (`*.ts, *.js...`) | Chỉ khi edit code |
| `06-testing` | **globs** (`*.test.*`) | Chỉ khi edit tests |
| `07-ai-workflow` | **description-based** | Hiếm khi cần |
| `08-spec-compliance` | **description-based** | Chỉ khi thay đổi spec |

### Kết quả

```
Trước (Old):  ████████████████████  ~2,000 dòng / tương tác
Sau (Custom): ████                  ~100–200 dòng / tương tác

Tiết kiệm: 60–70% tokens
```

`AGENTS.md` cũng được thiết kế lại: **~90 dòng router**, không lặp lại nội dung commands.

---

# Vấn đề #3: Team Collaboration

## Thực trạng với AIDLC Old

- **Không có** cơ chế phối hợp nhóm
- Không có spec locking → 2 người edit cùng spec
- Không có task ownership markers → tranh giành task
- Không có conflict playbook → xung đột mới tìm cách giải quyết
- Không mapping Scrum → không biết ai làm gì, khi nào

---

# Giải pháp: 3-Lane Team Workflow

```
 ┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐
 │  JP PM / BrSE    │  │    Tech Lead     │  │    Dev Team      │
 │  (Green Lane)    │  │    (Blue Lane)   │  │    (Yellow Lane) │
 ├──────────────────┤  ├──────────────────┤  ├──────────────────┤
 │ /aidlc-intake    │  │                  │  │                  │
 │ /aidlc-spec      │  │ /aidlc-confirm   │  │ /aidlc-build     │
 │ Customer Review  │  │ /aidlc-plan      │  │ /aidlc-fix       │
 │ UAT Validation   │  │ /aidlc-review    │  │ Create PR        │
 │                  │  │ /aidlc-release   │  │                  │
 │                  │  │ /aidlc-deploy    │  │                  │
 └──────────────────┘  └──────────────────┘  └──────────────────┘
```

### Các cơ chế bảo vệ

| Cơ chế | Mô tả |
|--------|-------|
| **Spec Lock** | `<!-- LOCKED_BY: @dev-a, 2026-04-17 -->` trong header file |
| **Task Ownership** | `[UNASSIGNED]` → `[IN_PROGRESS]` → `[DONE]` markers |
| **Morning Sync** | 15 phút: task hôm nay, blocker, file overlap |
| **Conflict Playbook** | 7 scenarios: file, spec, logic, sprint scope, blocked, divergence, multi-branch |
| **Logic Conflict** | Xung đột business rule ở 2 file khác nhau -- Git **không** phát hiện được |

---

# Vấn đề #4: Cài đặt phức tạp

## Thực trạng với AIDLC Old

```bash
# Phải copy thủ công TẤT CẢ -- không có lựa chọn
mkdir -p .cursor/commands/aidlc
cp /path/to/ai-dlc-sdd/cursor/commands/*.md .cursor/commands/aidlc/
cp /path/to/ai-dlc-sdd/cursor/AGENTS.md .cursor/
# + copy rules .mdc thủ công
# Không biết cần file nào, không cần file nào
```

- Cursor-only, không hỗ trợ platform khác
- Không có hướng dẫn chọn lọc

---

# Giải pháp: Tiered Installation

### 3 Tiers (75 files → 28–48 trong project)

| Tier | Nội dung | Số files |
|------|----------|----------|
| **Tier 1: Core** | 8 rules + 8 lifecycle commands + fix + AGENTS.md | 18 (luôn cài) |
| **Tier 2: Project Kit** | Templates, support commands, CI, skills | 10–35 (chọn lọc) |
| **Tier 3: Reference** | Docs, examples, READMEs | 18 (chỉ đọc) |

### Quy mô theo project

| Loại project | Tier 1 | Tier 2 | Tổng |
|-------------|--------|--------|------|
| Solo / light-spec | 18 | ~10 | **~28 files** |
| Small team / feature-spec | 18 | ~15 + 3 skills | **~36 files** |
| Multi-team / full-spec | 18 | ~27 + 3 skills | **~48 files** |

### `/aidlc-setup` tự động hoá

Hỏi → chọn mode → chọn features → copy đúng files cần thiết.

### Multi-platform

| Platform | Rules | Commands |
|----------|-------|----------|
| **Cursor** | `.cursor/rules/` | `.cursor/commands/` |
| **Google Antigravity** | `.agents/rules/` | `.agents/workflows/` |
| **Claude Code** | `.claude/rules/` | `.claude/commands/` |

---

# Vấn đề #5: Thiếu Lifecycle Management

## Các khả năng AIDLC Old **không có**

| Khả năng | Hậu quả |
|----------|---------|
| Hotfix process | Production incident → không có quy trình fast-track |
| UAT process | Không kiểm thử trước khi release |
| Release management | Không versioning, không CHANGELOG |
| Maintenance mode | Dự án ổn định vẫn phải chạy full flow |
| Project transfer | Bàn giao không có checklist → mất kiến thức |
| CI/CD templates | Mỗi project tự setup pipeline |
| Estimation | Không có framework ước lượng |

---

# Giải pháp: Bộ Lifecycle đầy đủ

| Khả năng | Command / Doc | Mô tả |
|----------|---------------|-------|
| **Hotfix** | `/aidlc-hotfix` | P0/P1 fast-track, Post-Incident Review |
| **UAT** | `docs/uat-process.md` | Checklist theo mode, bắt buộc cho full-spec |
| **Versioning** | `docs/versioning-guide.md` | SemVer, CHANGELOG, conventional commits |
| **Maintenance** | `docs/maintenance-guide.md` | Drift detection, khi nào re-enter delivery |
| **Transfer** | `/aidlc-transfer` | Handover checklist + verification |
| **CI/CD** | `docs/cicd-integration.md` | GitHub Actions + GitLab CI templates |
| **Sprint** | `docs/sprint-management.md` | AIDLC ↔ Scrum mapping, velocity |
| **Estimation** | `docs/estimation-guide.md` | T-shirt → story points → hours |
| **Issue Tracking** | `docs/issue-tracker-integration.md` | Jira / GitHub / Linear linkage |
| **Skills** | `docs/skills-lifecycle.md` | Common vs project skills, SIP, versioning |

> **12 guides + 4 worked examples** so với 4 docs (tiếng Nhật) của bản Old.

---

# Vấn đề #6: Artifact Organization

## AIDLC Old -- Tổ chức theo **design phase**

```
aidlc-docs/
├── requirements/          ← TẤT CẢ NFRs, risks chung 1 folder
├── design-artifacts/
│   ├── units/             ← TẤT CẢ units chung
│   ├── domain-models/     ← TẤT CẢ domain models chung
│   ├── adrs/              ← TẤT CẢ ADRs chung
│   └── logical-designs/   ← TẤT CẢ logical designs chung
└── plans/                 ← HÀNG CHỤC plan files lẫn lộn
    ├── inception_plan.md
    ├── code_generation_all-units_plan.md
    ├── code_generation_all_units_plan.md   ← trùng lặp!
    ├── modification_analysis_20250601.md
    ├── modification_analysis_20250615.md
    └── ... (tiếp tục phình to)
```

**Vấn đề**: Không biết artifact nào thuộc change nào. Tìm kiếm khó khăn.

---

# Giải pháp: Per-Change Folders

## AIDLC Custom -- Tổ chức theo **change**

```
aidlc-docs/changes/
├── active/
│   ├── CHG-001-login-rate-limit/    ← TẤT CẢ artifacts của change này
│   │   ├── proposal.md
│   │   ├── design.md
│   │   ├── tasks.md
│   │   └── ...
│   └── CHG-002-csv-export/          ← TẤT CẢ artifacts của change này
│       ├── proposal.md
│       └── ...
├── released/2026-Q1/                ← Đã release, shrunk
│   └── CHG-001-login-rate-limit/
│       ├── README.md
│       ├── proposal.md
│       └── release-note.md
└── archived/2026/                   ← Long-term, summary only
    └── CHG-001-login-rate-limit/
        └── summary.md
```

**Lợi ích**:
- Mỗi change là 1 folder **độc lập**: dễ tìm, dễ archive, dễ transfer
- Không còn `plans/` chứa hàng chục file lẫn lộn
- Lifecycle rõ ràng: active → released → archived

---

# So sánh tổng hợp

| Khía cạnh | AIDLC Old | AIDLC Custom |
|-----------|-----------|--------------|
| **Nguồn gốc** | ai-dlc-sdd (tamagoya) | AI-DLC + OpenSpec + Spec Kit |
| **Spec modes** | 1 (nặng) | **4 adaptive** (micro → full) |
| **Commands** | 16 (flat, 1 flow) | 17 (8 lifecycle + 9 support) |
| **Rules** | 4 `.mdc` all `alwaysApply` | 8 (1 alwaysApply + globs + description) |
| **Token cost** | ~2,000 dòng/tương tác | **~100–200 dòng** (-60–70%) |
| **Team workflow** | Không có | **3-lane + lock + conflict playbook** |
| **Artifact org** | Theo design phase | **Per-change folders** |
| **Knowledge lifecycle** | Tích luỹ mãi mãi | **3-tier shrink / archive** |
| **Installation** | Copy hết hoặc không | **Tiered (28–48 files)** + auto setup |
| **Platform** | Cursor only | **Cursor + Antigravity + Claude Code** |
| **Hotfix** | Không có | P0/P1 fast-track + PIR |
| **UAT** | Không có | Scaled by mode |
| **CI/CD** | Không có templates | **GH Actions + GitLab CI** templates |
| **Sprint mgmt** | Không có | Sprint = index, velocity tracking |
| **Docs / Guides** | 4 (tiếng Nhật) | **12 guides** (English) + examples |
| **Skills** | Planned, chưa implement | **3 Cursor skills** (TDD, review, archive) |
| **Ngôn ngữ** | Tiếng Nhật | **Tiếng Anh** (quốc tế hoá) |

---

# Distribution Models (Bonus)

| Model | Phù hợp | Rules update | Repo size |
|-------|---------|--------------|-----------|
| **A: Team Dashboard** | Team ≥ 2, Cursor | Auto-sync | Nhẹ |
| **B: All-in-project** | Solo / multi-platform | Manual (submodule) | Nặng hơn |
| **C: Manual** | Pilot / thử nghiệm | Manual copy | Nặng |

### Model A (khuyến nghị cho team):

```
Cursor Team Dashboard              Shared Git Repo (aidlc-custom/)
══════════════════════             ════════════════════════════════
 8 Rules (auto-sync)               Templates (copy khi setup)
 16 Commands (auto-sync)           Skills (copy khi setup)
  → update 1 lần, sync hết        Docs (đọc reference)
```

---

# Lộ trình triển khai

## Tuần 1: Foundation

- Chạy `/aidlc-setup` → cài Tier 1
- Dùng **micro** + **light-spec** cho 2–3 task đầu tiên
- Làm quen flow: intake → spec → confirm → build → review → release

## Tuần 2–3: Feature Work

- Nâng cấp lên **feature-spec** cho feature đầu tiên
- Thiết lập team workflow: branch naming, daily sync, spec locking
- Practice conflict playbook

## Tháng 2+: Full Capability

- Dùng **full-spec** cho feature phức tạp, multi-team
- Kích hoạt UAT, staging deploy, monitoring
- Thiết lập 3-tier storage + CI/CD pipeline

---

# Tips triển khai thành công

1. **Start micro, go heavy as needed** -- không dùng full-spec cho mọi thứ
2. **Confirmation gate là bất khả xâm phạm** -- không bao giờ skip `/aidlc-confirm`
3. **Spec trước code** -- viết proposal + design trước → giảm rework
4. **Templates là hướng dẫn, không phải nhà tù** -- tuỳ chỉnh theo change
5. **Traceability tạo giá trị lâu dài** -- linking artifacts tiết kiệm thời gian debug
6. **Daily sync 15 phút** -- phòng ngừa hơn chữa trị
7. **Đừng xoá kiến thức -- shrink nó** -- dùng 3-tier storage
8. **Sprint = index** -- sprint backlogs link đến changes, không copy spec

---

# Kết luận

## AIDLC Custom giải quyết 6 vấn đề cốt lõi

| # | Vấn đề | Giải pháp |
|---|--------|-----------|
| 1 | Repo bloat | 3-tier storage + shrink rules |
| 2 | Token overhead | Smart rule loading (-60–70%) |
| 3 | Thiếu team coordination | 3-lane workflow + lock + conflict playbook |
| 4 | Cài đặt phức tạp | Tiered installation + auto setup |
| 5 | Lifecycle gaps | Hotfix, UAT, transfer, maintenance, CI/CD |
| 6 | Artifact lẫn lộn | Per-change folders + clear lifecycle |

## Và bổ sung giá trị mới

- **4 spec modes** thay vì 1 flow cố định
- **Multi-platform** (Cursor, Antigravity, Claude Code)
- **3 Cursor Skills** (TDD, spec review, shrink/archive)
- **12 guides + 4 worked examples** thay vì 4 docs tiếng Nhật
- **Quốc tế hoá**: English-first, team đa quốc gia có thể dùng ngay

---

# Q&A

> **Tài liệu tham khảo**:
> - `README.md` -- Framework overview
> - `commands/README.md` -- Command catalog & flow diagrams
> - `AGENTS.md` -- Agent & role definitions
> - `docs/adoption-guide.md` -- Hướng dẫn triển khai chi tiết
> - `docs/developer-quickstart.md` -- Day-1 cheat sheet
> - `docs/spec-lifecycle.md` -- 3-Tier storage strategy

> **Credits**: Built on top of AI-DLC (AWS Labs), OpenSpec (Fission-AI), Spec Kit (GitHub), ai-dlc-sdd (tamagoya).
