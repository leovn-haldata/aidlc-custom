# Playbook triển khai SDD / AIDLC cho dự án multi-repo

**Phiên bản:** 1.0  
**Ngày:** 2026-05-06  
**Phạm vi:** `tv-app`, `tv-api`, `tv-backend`, `tv-sdd` / `aidlc-docs`  
**Mục tiêu:** Chuẩn hóa workflow phát triển bằng SDD/AIDLC cho dự án đã chạy production, tận dụng docs hiện có trong từng repo, giảm token khi dùng Cursor/Claude, và hỗ trợ nhiều member làm song song.

---

## Mục lục

1. Executive Summary
2. Bối cảnh hiện tại
3. Nguyên tắc thiết kế SDD/AIDLC cho multi-repo
4. Kiến trúc tài liệu đề xuất
5. Vai trò của `tv-sdd` và repo-local docs
6. Cách tận dụng docs hiện có trong từng repo
7. Lifecycle của một change/feature
8. Cấu trúc thư mục chi tiết
9. Workflow triển khai theo phase
10. Template chi tiết cho từng loại file
11. Ví dụ đầy đủ: Import CSV user data
12. Cách chia task cho từng repo/member
13. Cursor setup
14. Claude Code setup
15. Context loading và token optimization
16. Contract-first development
17. Review, QA, integration và release gate
18. Maintenance, governance và chống drift
19. Checklist triển khai ban đầu
20. Checklist cho từng feature
21. Prompt templates cho Cursor/Claude
22. Rủi ro thường gặp và cách xử lý
23. Kết luận

---

# 1. Executive Summary

Dự án hiện tại đã có nhiều repo và mỗi repo đã có docs riêng. Vì vậy không nên tạo một `tv-sdd` để copy lại toàn bộ tài liệu từ `tv-app`, `tv-api`, `tv-backend`.

Cách tối ưu là:

```txt
Repo docs hiện tại
  = source of truth chi tiết cho từng repo

tv-sdd / aidlc-docs
  = orchestration layer
  = index layer
  = feature/change layer
  = contract/integration layer
```

Nói ngắn gọn:

```txt
tv-sdd quản lý Why / What / Contract / Integration
repo docs quản lý How / Local detail / Implementation behavior
```

Với workflow này:

1. `tv-sdd` nhận yêu cầu feature/change.
2. `tv-sdd` xác định impacted domains, impacted repos, impacted docs.
3. `tv-sdd` tạo global feature spec và contract.
4. `tv-sdd` sinh task packet cho từng repo.
5. Mỗi member xử lý task packet trong repo mình.
6. Repo implementation được review độc lập.
7. Contract/integration test được chạy để ráp tổng thể.
8. Sau khi release, update docs/current-state cần thiết.

Mục tiêu không phải là tạo một bộ tài liệu lớn nhất có thể, mà là tạo một hệ thống tài liệu đủ chính xác, dễ truy xuất, ít duplicate, ít tốn token, và hỗ trợ AI làm việc đúng boundary.

---

# 2. Bối cảnh hiện tại

## 2.1 Repo hiện có

```txt
tv-app
tv-api
tv-backend
```

Các repo đã có docs nội bộ, ví dụ:

```txt
tv-api/docs/en/
  01.admin/
    01.auth/
    02.user-management/
    03.group-management/
    04.announcement-management/
    05.guide-management/
    00.overview.md

  02.general/
  03.widget/
```

Và các phần chi tiết khác như:

```txt
02.localdb-sync/
  00.overview.md
  01.missed-rankings-sync.md

03.crawler-integration/
  00.overview.md
  01.create-configs.md
  02.update-configs.md
  03.sync-failed.md
  04.sync-old-crawl-status.md
```

Đây là asset quan trọng. Không nên bỏ đi hoặc copy toàn bộ sang `tv-sdd`.

---

## 2.2 Vấn đề khi áp dụng AI/SDD trực tiếp vào multi-repo

Nếu mỗi feature đều yêu cầu AI đọc:

```txt
full tv-sdd
+ full tv-app docs
+ full tv-api docs
+ full tv-backend docs
+ source code liên quan
```

thì sẽ gặp các vấn đề:

- tốn token
- tốn thời gian reasoning
- AI dễ bị nhiễu vì quá nhiều context
- khó xác định repo ownership
- dễ sửa lan sang repo khác
- dễ drift giữa feature spec và repo implementation
- review khó vì không rõ contract nào là source of truth

Do đó cần một lớp điều phối trung gian.

---

# 3. Nguyên tắc thiết kế SDD/AIDLC cho multi-repo

## 3.1 Single source of truth theo cấp độ

Không có một file duy nhất làm source of truth cho mọi thứ. Thay vào đó cần phân cấp:

| Cấp độ | Source of truth | Nội dung |
|---|---|---|
| Business / Product | `tv-sdd/features/<feature>/requirements.md` | Vì sao làm, user muốn gì, acceptance criteria |
| Cross-repo flow | `tv-sdd/features/<feature>/overview.md` hoặc `integration-plan.md` | Luồng tổng thể qua nhiều repo |
| Contract | `tv-sdd/features/<feature>/contracts.md` hoặc `tv-sdd/current-state/contracts/` | API/event/data contract |
| Local behavior | Repo docs hiện có | Chi tiết behavior/module trong repo |
| Implementation | Source code từng repo | Code thực tế |
| Repo task | `task-packets/<repo>.md` hoặc repo-local `aidlc-docs/changes/...` | Scope công việc của repo đó |

---

## 3.2 Không duplicate docs

Không viết lại đầy đủ docs repo trong `tv-sdd`.

Sai:

```txt
tv-sdd/current-state/tv-api/full-user-management.md
```

rồi copy toàn bộ nội dung từ:

```txt
tv-api/docs/en/01.admin/02.user-management/
```

Đúng:

```txt
tv-sdd/current-state/indexes/tv-api-doc-index.md
tv-sdd/features/import-csv-user-data/affected-docs.md
```

Trong đó chỉ reference tới docs gốc:

```md
## tv-api
Read first:
- tv-api/docs/en/01.admin/02.user-management/00.overview.md
- tv-api/docs/en/01.admin/02.user-management/01.create-user.md
```

---

## 3.3 Feature-centric cho change, domain/repo-centric cho current state

Nên chia như sau:

```txt
features/
  = feature/change-centric

current-state/
  = architecture/domain/repo/contract-centric
```

Không nên chia mọi thứ theo feature, vì current-state dùng lại giữa nhiều feature. Nếu current-state cũng bị nhét vào từng feature, sẽ bị duplicate và drift.

---

## 3.4 Task packet phải nhỏ và rõ boundary

Mỗi repo nhận một task packet riêng.

Task packet không viết lại global spec, chỉ chứa:

- link tới global spec
- scope repo này
- out of scope
- contract liên quan
- affected local docs
- expected files/modules
- test plan
- definition of done

---

## 3.5 AI không được tự quyết định architecture cross-repo

AI có thể hỗ trợ:

- phân tích impact
- generate task packet
- implement local code
- viết test
- review diff
- update docs

Nhưng team/spec owner nên quyết định:

- API shape
- event contract
- breaking change
- migration strategy
- ownership giữa repo
- release sequencing

---

# 4. Kiến trúc tài liệu đề xuất

## 4.1 Overview

```txt
tv-sdd/
  aidlc-docs/
    changes/
    current-state/
    design-artifacts/
    operations/
    plans/
    sprints/
    templates/
    indexes/
```

Nếu bạn đã có folder tên `aidlc-docs`, có thể dùng trực tiếp thay vì tạo thêm `tv-sdd/features`.

---

## 4.2 Vai trò các folder chính

```txt
aidlc-docs/changes/
```

Quản lý các change/feature theo lifecycle.

```txt
aidlc-docs/current-state/
```

Mô tả trạng thái hiện tại của hệ thống ở mức index/domain/contract, không copy chi tiết repo docs.

```txt
aidlc-docs/design-artifacts/
```

Chứa contract, diagram, ADR, schema, flow tổng thể.

```txt
aidlc-docs/operations/
```

Chứa guideline vận hành, CI/CD, deploy notes, rollback notes, monitoring notes.

```txt
aidlc-docs/plans/
```

Chứa roadmap, milestone, rollout plan.

```txt
aidlc-docs/sprints/
```

Chứa sprint-level planning, feature assignment, status.

```txt
aidlc-docs/templates/
```

Chứa template chuẩn để tạo spec/task/review/release.

```txt
aidlc-docs/indexes/
```

Chứa index map tới docs hiện có trong từng repo.

---

# 5. Vai trò của `tv-sdd` và repo-local docs

## 5.1 `tv-sdd` / central AIDLC docs

`tv-sdd` chịu trách nhiệm:

- nhận yêu cầu
- xác định impacted domains
- xác định impacted repos
- xác định affected docs hiện có
- tạo feature spec
- tạo contract
- tạo task packet
- theo dõi trạng thái cross-repo
- định nghĩa integration test plan
- lưu decision log/ADR
- release checklist
- archive change sau khi hoàn tất

`tv-sdd` không nên chứa:

- toàn bộ code
- toàn bộ docs chi tiết của từng repo
- implementation detail quá sâu
- task coding nhỏ từng file nếu không cần thiết

---

## 5.2 Repo-local docs

Mỗi repo tiếp tục giữ docs chi tiết hiện có.

Ví dụ `tv-api` vẫn giữ:

```txt
tv-api/docs/en/01.admin/02.user-management/
tv-api/docs/en/03.widget/
tv-api/docs/en/02.localdb-sync/
```

Repo-local docs chịu trách nhiệm:

- mô tả module/local behavior
- local API behavior nếu thuộc repo đó
- command/test của repo
- local architecture
- patterns/conventions
- troubleshooting
- docs chi tiết cho module

---

## 5.3 Repo-local AIDLC task

Ngoài docs hiện có, mỗi repo có thể có thêm:

```txt
aidlc-docs/changes/active/<change-id>.md
```

hoặc:

```txt
aidlc-docs/changes/active/<change-id>/
  task.md
  plan.md
  status.md
```

Nội dung này chỉ là implementation slice của repo đó, không phải full SDD.

---

# 6. Cách tận dụng docs hiện có trong từng repo

## 6.1 Không chuyển docs, chỉ index docs

Tạo file index ở central:

```txt
tv-sdd/aidlc-docs/indexes/repo-doc-index.md
```

Ví dụ:

```md
# Repo Doc Index

## tv-api

### Admin
- Auth: tv-api/docs/en/01.admin/01.auth/00.overview.md
- User management: tv-api/docs/en/01.admin/02.user-management/00.overview.md
- Group management: tv-api/docs/en/01.admin/03.group-management/00.overview.md
- Announcement management: tv-api/docs/en/01.admin/04.announcement-management/00.overview.md
- Guide management: tv-api/docs/en/01.admin/05.guide-management/00.overview.md

### Sync
- Local DB sync: tv-api/docs/en/02.localdb-sync/00.overview.md
- Crawler integration: tv-api/docs/en/03.crawler-integration/00.overview.md
```

---

## 6.2 Tạo domain index

```txt
tv-sdd/aidlc-docs/indexes/domain-index.md
```

Ví dụ:

```md
# Domain Index

## User

Related repos:
- tv-app
- tv-api
- tv-backend

Related docs:
- tv-api/docs/en/01.admin/02.user-management/00.overview.md
- tv-app/docs/en/admin/user-management/00.overview.md
- tv-backend/docs/en/jobs/user-notification/00.overview.md

Related contracts:
- GET /users
- GET /users/{id}
- POST /users
- PATCH /users/{id}

Related jobs/events:
- user-notification-job
```

---

## 6.3 Tạo affected docs cho từng feature

Mỗi feature cần file:

```txt
affected-docs.md
```

Mục tiêu là chỉ cho AI/member đọc đúng tài liệu cần thiết.

Ví dụ:

```md
# Affected Docs - Import CSV User Data

## Read first

### tv-api
- tv-api/docs/en/01.admin/02.user-management/00.overview.md
- tv-api/docs/en/01.admin/02.user-management/01.create-user.md

### tv-app
- tv-app/docs/en/admin/user-management/00.overview.md

### tv-backend
- tv-backend/docs/en/jobs/notification/00.overview.md

## Read only if needed

### tv-api
- tv-api/docs/en/01.admin/01.auth/00.overview.md

### tv-backend
- tv-backend/docs/en/jobs/retry-policy.md

## Do not read unless explicitly requested

- unrelated billing docs
- unrelated crawler docs
- unrelated widget docs
```

---

## 6.4 Khi nào update docs repo?

Update repo docs khi:

- behavior local thay đổi
- thêm API/module trong repo
- thêm job/event trong repo
- thêm validation/error behavior trong repo
- thay đổi command/test/deploy flow trong repo

Không cần update repo docs khi:

- chỉ thay đổi global planning
- chỉ thay đổi task status
- chỉ thay đổi cross-repo integration plan nhưng local behavior không đổi

---

# 7. Lifecycle của một change/feature

## 7.1 Trạng thái change

```txt
changes/
  active/
  released/
  archived/
```

Ý nghĩa:

| State | Ý nghĩa |
|---|---|
| active | đang phân tích/implement/review |
| released | đã release và còn giá trị tham chiếu |
| archived | đã cũ, không còn active reference |

---

## 7.2 Flow tổng thể

```txt
1. Intake
2. Brownfield discovery
3. Current-state summary nếu cần
4. Feature spec
5. Contract design
6. Task packet generation
7. Repo-local planning
8. Repo implementation
9. Repo review
10. Contract validation
11. Integration test
12. Release
13. Current-state update
14. Move released/archive
```

---

## 7.3 Gate chi tiết

### Gate 0 — Intake accepted

Đầu ra cần có:

- problem statement
- business goal
- impacted domain dự kiến
- owner
- target release nếu có

---

### Gate 1 — Brownfield discovery completed

Đầu ra cần có:

- affected docs
- affected repos
- existing capabilities
- reuse candidates
- risks/unknowns

---

### Gate 2 — Contract frozen

Đầu ra cần có:

- API/event/data contract
- backward compatibility decision
- error format
- versioning nếu cần
- consumer/provider mapping

---

### Gate 3 — Repo task packets approved

Đầu ra cần có:

- `task-packets/tv-app.md`
- `task-packets/tv-api.md`
- `task-packets/tv-backend.md`
- owner cho từng repo
- local test plan

---

### Gate 4 — Local implementation done

Mỗi repo cần có:

- code done
- local tests pass
- repo docs updated nếu cần
- local review done

---

### Gate 5 — Integration done

Cần có:

- contract tests pass
- integration tests pass
- manual QA nếu cần
- migration/deploy plan verified

---

### Gate 6 — Released and documented

Cần có:

- release note
- current-state updated
- feature moved to released
- known issues documented

---

# 8. Cấu trúc thư mục chi tiết

## 8.1 Central `tv-sdd` / `aidlc-docs`

```txt
tv-sdd/
  aidlc-docs/
    README.md

    changes/
      active/
        import-csv-user-data/
          00.overview.md
          01.requirements.md
          02.brownfield-discovery.md
          03.affected-docs.md
          04.contracts.md
          05.implementation-plan.md
          06.integration-test-plan.md
          07.status.md
          08.release-plan.md
          task-packets/
            tv-app.md
            tv-api.md
            tv-backend.md

      released/
      archived/

    current-state/
      architecture/
        system-overview.md
        repo-responsibility.md
        dependency-map.md

      domains/
        user.md
        auth.md
        notification.md

      contracts/
        api-index.md
        event-index.md
        data-schema-index.md

      flows/
        login-flow.md
        user-management-flow.md
        notification-flow.md

    indexes/
      repo-doc-index.md
      domain-index.md
      feature-index.md
      contract-index.md

    design-artifacts/
      adrs/
      diagrams/
      schemas/
      openapi/
      events/

    operations/
      deploy.md
      rollback.md
      monitoring.md
      incident-response.md

    templates/
      feature-overview.template.md
      requirements.template.md
      brownfield-discovery.template.md
      affected-docs.template.md
      contracts.template.md
      task-packet.template.md
      integration-test-plan.template.md
      status.template.md
      review.template.md
      release-plan.template.md
      adr.template.md
```

---

## 8.2 Repo-local structure

Mỗi repo có thể thêm phần AIDLC nhỏ:

```txt
tv-api/
  docs/
    en/
      ...

  aidlc-docs/
    README.md
    changes/
      active/
        import-csv-user-data.md
      released/
      archived/

  .cursor/
    rules/
      repo-overview.mdc
      api-patterns.mdc
      testing.mdc
      docs-policy.mdc

  .claude/
    rules/
      api-paths.md
      testing.md

  CLAUDE.md
  AGENTS.md
```

Tương tự cho `tv-app` và `tv-backend`.

---

## 8.3 Có nên copy task packet về repo không?

Có 2 option.

### Option A — Central only

Task packet nằm trong `tv-sdd`:

```txt
tv-sdd/aidlc-docs/changes/active/import-csv-user-data/task-packets/tv-api.md
```

Member làm `tv-api` mở file này từ central.

Ưu điểm:
- không duplicate
- dễ quản lý
- ít drift

Nhược điểm:
- khi mở riêng repo có thể bất tiện
- Cursor/Claude trong repo có thể không thấy file nếu không add-dir hoặc không mở workspace nhiều folder

---

### Option B — Copy/link minimal task về repo

Repo có:

```txt
tv-api/aidlc-docs/changes/active/import-csv-user-data.md
```

Nội dung là bản rút gọn hoặc reference:

```md
# Import CSV User Data - tv-api Task

Global source:
- tv-sdd/aidlc-docs/changes/active/import-csv-user-data/

This repo scope:
- implement upload endpoint
- implement import status endpoint
- add validation tests

Do not edit:
- tv-app
- tv-backend
```

Ưu điểm:
- mỗi repo tự chạy độc lập tốt
- member dễ làm việc
- Cursor/Claude chỉ cần mở repo hiện tại

Nhược điểm:
- cần sync nếu task thay đổi

Khuyến nghị ban đầu: **Option B với nội dung ngắn**, vì team/member dễ áp dụng hơn. Khi workflow ổn, có thể chuyển sang symlink hoặc automation.

---

# 9. Workflow triển khai theo phase

## Phase 0 — Inventory docs hiện có

Mục tiêu:

- biết repo nào có docs gì
- tạo index ban đầu
- không viết lại docs

Output:

```txt
aidlc-docs/indexes/repo-doc-index.md
aidlc-docs/indexes/domain-index.md
aidlc-docs/current-state/architecture/repo-responsibility.md
```

Checklist:

- [ ] scan docs của `tv-app`
- [ ] scan docs của `tv-api`
- [ ] scan docs của `tv-backend`
- [ ] map docs theo domain
- [ ] map domain theo repo ownership
- [ ] identify missing docs quan trọng

---

## Phase 1 — Tạo central SDD skeleton

Mục tiêu:

- có folder structure chuẩn
- có template chuẩn
- có quy trình active/released/archived

Output:

```txt
aidlc-docs/changes/
aidlc-docs/templates/
aidlc-docs/indexes/
aidlc-docs/current-state/
```

Checklist:

- [ ] tạo templates
- [ ] tạo README
- [ ] tạo change lifecycle
- [ ] tạo status format
- [ ] tạo naming convention

---

## Phase 2 — Pilot với một feature thật

Nên dùng feature có đủ cross-repo impact, ví dụ:

```txt
Import CSV user data
```

Mục tiêu:

- test workflow end-to-end
- đo xem AI tốn context ở đâu
- tìm chỗ docs thiếu
- chuẩn hóa task packet

Output:

```txt
changes/active/import-csv-user-data/
```

Checklist:

- [ ] intake
- [ ] brownfield discovery
- [ ] affected docs
- [ ] contracts
- [ ] task packets
- [ ] repo implementation
- [ ] integration test
- [ ] release
- [ ] update current-state

---

## Phase 3 — Standardize

Sau 2-3 feature thật, chuẩn hóa:

- template
- naming
- review checklist
- task packet format
- prompt commands
- AI rules
- CI validation

---

## Phase 4 — Automation

Chỉ làm automation sau khi workflow thủ công đã ổn.

Có thể automate:

- generate affected-docs từ domain-index
- generate task-packets từ feature spec
- validate contract references
- stale docs detection
- change status dashboard
- PR checklist generation

---

# 10. Template chi tiết cho từng loại file

## 10.1 `00.overview.md`

```md
# <Feature Name>

## Change ID
<kebab-case-id>

## Status
active | released | archived

## Owner
<Name/team>

## Summary
1-3 câu mô tả change.

## Business goal
Vì sao cần làm?

## User impact
Ai bị ảnh hưởng? Admin/user/internal team?

## System impact
Feature này chạm vào repo/domain nào?

## Affected repos
- tv-app
- tv-api
- tv-backend

## Affected domains
- user
- notification

## High-level flow
1. ...
2. ...
3. ...

## Non-goals
- Không làm ...
- Không thay đổi ...

## Links
- Requirements: ./01.requirements.md
- Brownfield discovery: ./02.brownfield-discovery.md
- Contracts: ./04.contracts.md
- Integration plan: ./06.integration-test-plan.md
```

---

## 10.2 `01.requirements.md`

```md
# Requirements - <Feature Name>

## Problem statement
Mô tả vấn đề hiện tại.

## Business requirements

### R1
As a <role>, I want <capability>, so that <benefit>.

Acceptance criteria:
- [ ] ...
- [ ] ...

### R2
...

## Functional requirements
- FR1:
- FR2:

## Non-functional requirements
- Performance:
- Security:
- Audit:
- Observability:
- Backward compatibility:

## Edge cases
- invalid input
- duplicate data
- permission denied
- partial failure
- retry

## Out of scope
- ...
```

---

## 10.3 `02.brownfield-discovery.md`

```md
# Brownfield Discovery - <Feature Name>

## Goal
Xác định hệ thống hiện tại đã có gì và cần reuse/chỉnh sửa/tạo mới gì.

## Current capabilities
- Existing user creation:
- Existing user list:
- Existing notification job:

## Existing APIs
| API | Repo | Doc | Notes |
|---|---|---|---|
| GET /users | tv-api | ... | existing |
| POST /users | tv-api | ... | existing |

## Existing jobs/events
| Job/Event | Repo | Doc | Notes |
|---|---|---|---|

## Existing data model
| Entity | Fields | Notes |
|---|---|---|

## Reuse candidates
- existing validation pattern
- existing notification job framework
- existing user list UI pattern

## Gaps
- no bulk import endpoint
- no import status tracking
- no failed row report

## Risks
- duplicate email handling
- large CSV size
- partial failure behavior
- notification spam

## Unknowns
- max CSV size?
- sync vs async import?
- who receives notification?
```

---

## 10.4 `03.affected-docs.md`

```md
# Affected Docs - <Feature Name>

## Read first

### tv-app
- path/to/doc.md
Reason: ...

### tv-api
- path/to/doc.md
Reason: ...

### tv-backend
- path/to/doc.md
Reason: ...

## Read if needed
- ...

## Do not read by default
- ...

## Missing docs discovered
- [ ] ...
```

---

## 10.5 `04.contracts.md`

```md
# Contracts - <Feature Name>

## Contract status
draft | frozen | changed-after-freeze

## API contracts

### POST /users/import

Provider:
- tv-api

Consumers:
- tv-app

Request:
```json
{
  "file": "<csv multipart file>",
  "notifyUsers": true
}
```

Response:
```json
{
  "importId": "imp_123",
  "status": "queued"
}
```

Errors:
```json
{
  "code": "CSV_INVALID_HEADER",
  "message": "CSV header is invalid",
  "details": {
    "missingColumns": ["email"]
  }
}
```

### GET /imports/{id}/status

Response:
```json
{
  "importId": "imp_123",
  "status": "processing",
  "totalRows": 100,
  "processedRows": 60,
  "successRows": 55,
  "failedRows": 5
}
```

## Event/job contracts

### user-import-created

Producer:
- tv-api

Consumer:
- tv-backend

Payload:
```json
{
  "importId": "imp_123",
  "fileLocation": "storage://...",
  "requestedBy": "admin_123",
  "notifyUsers": true
}
```

## Compatibility notes
- Existing user APIs must remain backward compatible.
- New import endpoints are additive.

## Contract change process
Sau khi status là frozen, mọi thay đổi phải update:
- this file
- affected task packets
- affected tests
- status.md
```

---

## 10.6 `05.implementation-plan.md`

```md
# Implementation Plan - <Feature Name>

## Recommended order

1. Freeze contract
2. Implement tv-api contract skeleton
3. Implement tv-backend job logic
4. Implement tv-app UI against mock/generated client
5. Run contract/integration tests
6. Release

## Repo dependency

| Repo | Depends on | Can start when |
|---|---|---|
| tv-app | API contract | contract frozen |
| tv-api | contract | immediately after contract draft |
| tv-backend | job/event contract | contract frozen |

## Rollout strategy
- feature flag?
- admin-only?
- limited CSV size in phase 1?

## Migration
- new tables?
- new queues?
- new storage bucket?
```

---

## 10.7 `06.integration-test-plan.md`

```md
# Integration Test Plan - <Feature Name>

## Test environment
- environment:
- required data:
- required services:

## Test cases

### IT1 - Successful import
Steps:
1. Upload CSV with valid users
2. Wait for import completed
3. Verify users appear in list
4. Verify user detail page works
5. Verify notification job executed

Expected:
- status = completed
- successRows = totalRows
- no failedRows

### IT2 - Invalid CSV header
Expected:
- API returns validation error
- no job created

### IT3 - Partial failure
Expected:
- valid rows imported
- invalid rows recorded
- status = completed_with_errors

### IT4 - Permission denied
Expected:
- non-admin cannot import

### IT5 - Retry behavior
Expected:
- failed notification retry follows policy

## Exit criteria
- [ ] all critical integration tests pass
- [ ] no contract mismatch
- [ ] rollback plan ready
```

---

## 10.8 `07.status.md`

```md
# Status - <Feature Name>

## Overall status
draft | planning | in-progress | integration | released | blocked

## Contract
- [ ] drafted
- [ ] reviewed
- [ ] frozen
- [ ] changed after freeze? no

## Repo status

| Repo | Owner | Status | PR | Notes |
|---|---|---|---|---|
| tv-app |  | not-started |  |  |
| tv-api |  | not-started |  |  |
| tv-backend |  | not-started |  |  |

## Open questions
- [ ] ...

## Blockers
- [ ] ...

## Decisions
- ...
```

---

## 10.9 `task-packets/<repo>.md`

```md
# <Feature Name> - <Repo> Task Packet

## Source of truth
Central spec:
- tv-sdd/aidlc-docs/changes/active/<feature-id>/

Read in order:
1. 00.overview.md
2. 03.affected-docs.md
3. 04.contracts.md
4. this task packet
5. affected local docs only

## Repo scope
This repo is responsible for:
- ...

## Out of scope
This repo must not:
- modify other repo behavior
- change contract without approval
- implement unrelated refactor

## Relevant contracts
- ...

## Affected local docs
- ...

## Expected code areas
- ...

## Implementation notes
- ...

## Tests required
- ...

## Definition of done
- [ ] implementation completed
- [ ] tests passed
- [ ] docs updated if needed
- [ ] no contract drift
- [ ] PR summary includes SDD references

## Questions for spec owner
- ...
```

---

# 11. Ví dụ đầy đủ: Import CSV user data

## 11.1 Central overview

```md
# Import CSV User Data

## Change ID
import-csv-user-data

## Summary
Cho phép admin upload CSV để import user hàng loạt, xem trạng thái import, xem user list/detail, và trigger batch notification tới các user được import.

## Affected repos
- tv-app
- tv-api
- tv-backend

## Affected domains
- user
- admin
- notification
- import/job processing

## High-level flow
1. Admin mở màn hình User Management trong tv-app.
2. Admin upload CSV.
3. tv-app gọi tv-api `POST /users/import`.
4. tv-api validate request, tạo import record, enqueue job/event.
5. tv-backend xử lý CSV, tạo/update user, gửi notification nếu được bật.
6. tv-api expose import status.
7. tv-app hiển thị trạng thái import và user list/detail.

## Non-goals
- Không thay đổi auth model.
- Không thay đổi toàn bộ user CRUD hiện tại.
- Không build generic CSV import framework cho mọi domain trong phase đầu.
```

---

## 11.2 Affected docs

```md
# Affected Docs - Import CSV User Data

## tv-api

Read first:
- tv-api/docs/en/01.admin/02.user-management/00.overview.md
- tv-api/docs/en/01.admin/02.user-management/01.create-user.md
- tv-api/docs/en/01.admin/02.user-management/02.update-user.md

Read if needed:
- tv-api/docs/en/01.admin/01.auth/00.overview.md
- tv-api/docs/en/07.notification/00.overview.md

## tv-app

Read first:
- tv-app/docs/en/admin/user-management/00.overview.md
- tv-app/docs/en/admin/user-management/user-list.md
- tv-app/docs/en/admin/user-management/user-detail.md

## tv-backend

Read first:
- tv-backend/docs/en/jobs/notification/00.overview.md
- tv-backend/docs/en/jobs/retry-policy.md

Read if needed:
- tv-backend/docs/en/storage/file-processing.md
```

---

## 11.3 Contract draft

```md
# Contracts - Import CSV User Data

## POST /users/import

Type:
- multipart/form-data

Fields:
- file: CSV file
- notifyUsers: boolean

CSV columns:
- email: required
- name: required
- groupCode: optional
- status: optional

Response 202:
```json
{
  "importId": "imp_123",
  "status": "queued"
}
```

## GET /imports/{id}/status

Response 200:
```json
{
  "importId": "imp_123",
  "status": "processing",
  "totalRows": 100,
  "processedRows": 80,
  "successRows": 75,
  "failedRows": 5,
  "createdAt": "2026-05-06T10:00:00Z",
  "updatedAt": "2026-05-06T10:02:00Z"
}
```

## Import statuses
- queued
- processing
- completed
- completed_with_errors
- failed

## Error codes
- CSV_INVALID_HEADER
- CSV_FILE_TOO_LARGE
- CSV_EMPTY
- CSV_ROW_INVALID
- IMPORT_NOT_FOUND
- PERMISSION_DENIED
```

---

## 11.4 tv-app task packet

```md
# Import CSV User Data - tv-app

## Source of truth
- tv-sdd/aidlc-docs/changes/active/import-csv-user-data/

## Scope
Implement frontend only:
- CSV upload UI
- upload progress/state
- import status display
- user list page update if needed
- user detail page update if needed
- error display from API

## Out of scope
- CSV parsing on server
- DB migration
- notification job logic
- API response schema change

## Relevant contracts
- POST /users/import
- GET /imports/{id}/status
- GET /users
- GET /users/{id}

## Affected local docs
- tv-app/docs/en/admin/user-management/00.overview.md
- tv-app/docs/en/admin/user-management/user-list.md
- tv-app/docs/en/admin/user-management/user-detail.md

## Implementation notes
- Prefer existing form/upload component.
- Use generated API client or existing API call pattern.
- Do not hardcode API response shape outside typed client/model.
- Show API error `code` and user-friendly message.

## Tests
- upload form renders
- file type validation
- API success state
- API error state
- import status polling/display
- user list still renders

## Definition of done
- [ ] UI implemented
- [ ] local tests pass
- [ ] no contract change
- [ ] docs updated if UI behavior changed
```

---

## 11.5 tv-api task packet

```md
# Import CSV User Data - tv-api

## Source of truth
- tv-sdd/aidlc-docs/changes/active/import-csv-user-data/

## Scope
Implement API layer:
- POST /users/import
- GET /imports/{id}/status
- validation
- permission check
- create import record
- emit job/event for tv-backend

## Out of scope
- frontend UI
- actual notification delivery
- worker internals

## Relevant contracts
- API contract in 04.contracts.md
- job/event payload `user-import-created`

## Affected local docs
- tv-api/docs/en/01.admin/02.user-management/00.overview.md
- tv-api/docs/en/01.admin/02.user-management/01.create-user.md

## Implementation notes
- Follow existing admin permission pattern.
- Use existing error response format.
- Do not parse/process full CSV synchronously if design says async.
- Store enough metadata for status endpoint.

## Tests
- contract test
- permission test
- invalid CSV metadata test
- enqueue job/event test
- status endpoint test

## Definition of done
- [ ] endpoint implemented
- [ ] tests pass
- [ ] API docs updated
- [ ] contract matches central spec
```

---

## 11.6 tv-backend task packet

```md
# Import CSV User Data - tv-backend

## Source of truth
- tv-sdd/aidlc-docs/changes/active/import-csv-user-data/

## Scope
Implement backend processing:
- consume `user-import-created`
- load CSV
- validate rows
- create/update users
- record failed rows
- update import status
- send notification batch if `notifyUsers=true`

## Out of scope
- upload UI
- public API response shape
- user list/detail UI

## Relevant contracts
- job/event contract in 04.contracts.md
- import status model

## Affected local docs
- tv-backend/docs/en/jobs/notification/00.overview.md
- tv-backend/docs/en/jobs/retry-policy.md

## Implementation notes
- Job must be idempotent.
- Failed rows must be recorded.
- Notification retry must follow existing policy.
- Avoid duplicate notification on retry.

## Tests
- valid CSV import
- invalid row handling
- duplicate email handling
- idempotency test
- notification triggered
- notification retry/failure

## Definition of done
- [ ] worker implemented
- [ ] tests pass
- [ ] docs updated
- [ ] no contract drift
```

---

# 12. Cách chia task cho từng repo/member

## 12.1 Ownership model

| Role | Responsibility |
|---|---|
| Feature owner | chịu trách nhiệm global spec, status, final integration |
| Spec owner | viết/duyệt requirements và contract |
| Repo owner | triển khai task trong repo |
| Reviewer | review code/spec trong repo |
| Integration owner | chạy integration test, verify cross-repo behavior |

---

## 12.2 Assignment example

```txt
Feature: Import CSV User Data

Feature owner:
- Tech lead / BA / Senior dev

tv-app owner:
- Frontend member

tv-api owner:
- API member

tv-backend owner:
- Backend/job member

Integration owner:
- QA / Tech lead / rotating owner
```

---

## 12.3 Parallel workflow

Sau khi contract frozen:

```txt
tv-app
  build UI with mock/generated API

tv-api
  implement endpoints and contract tests

tv-backend
  implement job against event payload
```

Không cần đợi repo này xong repo kia mới bắt đầu, miễn là contract rõ.

---

## 12.4 Handoff rule

Mỗi repo PR cần include:

```md
## SDD References
- Central spec:
- Task packet:
- Contract:

## Scope completed
- ...

## Tests run
- ...

## Contract changes
- none / list changes

## Docs updated
- yes/no
```

---

# 13. Cursor setup

## 13.1 Mục tiêu setup

Cursor cần được hướng dẫn ở 3 cấp:

1. Global team/project behavior
2. Repo-specific behavior
3. Path/domain-specific behavior

Không nên nhét mọi rule vào một file dài.

---

## 13.2 Cấu trúc đề xuất

```txt
tv-api/
  AGENTS.md
  .cursor/
    rules/
      repo-overview.mdc
      api-contracts.mdc
      testing.mdc
      docs-policy.mdc
```

Ví dụ `repo-overview.mdc`:

```md
---
description: tv-api repository overview and boundaries
alwaysApply: true
---

# tv-api Rules

- This repo owns public/admin API behavior.
- Do not modify frontend UI.
- Do not implement worker internals unless explicitly requested.
- Contract changes must update central SDD contract first.
```

Ví dụ `api-contracts.mdc`:

```md
---
description: API contract rules for tv-api
globs:
  - "app/**/*.php"
  - "routes/**/*.php"
  - "docs/**/*.md"
---

# API Contract Rules

- Follow existing API response format.
- Use existing error code conventions.
- Do not change response schema without updating:
  - central contract
  - API docs
  - tests
```

---

## 13.3 Cursor working mode

Khuyến nghị khi implement repo task:

1. Mở repo riêng trong Cursor.
2. Attach/quote task packet tương ứng.
3. Yêu cầu Agent tạo plan trước.
4. Chỉ cho code sau khi review plan.
5. Chạy tests/lint.
6. Yêu cầu Agent summarize diff theo SDD checklist.

Prompt mẫu:

```txt
Read the repo task packet below and the referenced local docs only.
Do not edit files yet.

First produce:
1. implementation plan
2. affected files
3. risks
4. test plan
5. questions/blockers

Then wait for approval.
```

---

# 14. Claude Code setup

## 14.1 File nên có

```txt
CLAUDE.md
.claude/rules/
```

Ví dụ:

```txt
tv-api/
  CLAUDE.md
  .claude/
    rules/
      api-contracts.md
      testing.md
      docs.md
```

---

## 14.2 `CLAUDE.md` nên ngắn

Ví dụ:

```md
# tv-api Claude Instructions

## Repository boundary
- This repo owns API layer only.
- Do not modify tv-app or tv-backend.
- Do not change central contracts without explicit approval.

## Development workflow
- Always inspect the task packet first.
- Produce a plan before editing.
- Prefer small, reviewable changes.
- Run relevant tests before final summary.

## Documentation
- Update repo docs when behavior changes.
- Link PRs to central SDD change ID.
```

---

## 14.3 Claude path-scoped rules

Ví dụ:

```md
# .claude/rules/api-contracts.md

Applies when working on API routes/controllers/resources.

Rules:
- Preserve backward compatibility unless the task explicitly allows breaking changes.
- Use existing response wrapper.
- Add tests for every new endpoint.
- Update docs when adding/changing public API.
```

---

## 14.4 Khi làm việc multi-repo với Claude

Có 2 cách:

### Cách 1 — Một session cho một repo

Phù hợp nhất cho team.

```txt
Claude session 1: tv-api
Claude session 2: tv-app
Claude session 3: tv-backend
```

Mỗi session nhận đúng task packet của repo đó.

### Cách 2 — Một session integration

Chỉ dùng khi đã xong local implementation.

Session này đọc:

- central overview
- contracts
- status
- PR summaries
- integration test plan

Không đọc toàn bộ code cả 3 repo nếu chưa cần.

---

# 15. Context loading và token optimization

## 15.1 Context layers

Chia context thành nhiều layer:

| Layer | Nội dung | Khi nào đọc |
|---|---|---|
| L0 | task request hiện tại | luôn đọc |
| L1 | feature overview/requirements | luôn đọc khi làm feature |
| L2 | contract/affected-docs | luôn đọc khi implement |
| L3 | local repo docs liên quan | chỉ đọc theo affected-docs |
| L4 | source code liên quan | chỉ đọc khi edit |
| L5 | toàn bộ repo docs/code | tránh đọc, chỉ dùng khi discovery đặc biệt |

---

## 15.2 Context protocol cho AI

Mỗi task packet nên ghi rõ:

```md
## Context loading order

1. Read this task packet.
2. Read central overview.
3. Read contracts.
4. Read affected local docs listed below.
5. Search code only within expected code areas first.
6. Ask before expanding scope.
```

---

## 15.3 Token budget gợi ý

Không cần đo chính xác tuyệt đối, nhưng nên đặt guardrail:

| Stage | Context budget mong muốn |
|---|---|
| Intake | nhỏ |
| Brownfield discovery | vừa |
| Repo implementation | nhỏ-vừa |
| Integration review | vừa |
| Full architecture review | lớn, hiếm khi dùng |

Rule thực tế:

```txt
Nếu một prompt cần đọc quá nhiều file, hãy tạo index hoặc summary trước.
```

---

## 15.4 Anti-pattern gây tốn token

Tránh:

```txt
"Read all docs and implement this feature"
```

Thay bằng:

```txt
"Read affected-docs.md, then read only the docs listed under Read first for tv-api."
```

Tránh:

```txt
"Analyze all repos"
```

Thay bằng:

```txt
"Analyze impacted docs for user domain only."
```

---

# 16. Contract-first development

## 16.1 Vì sao contract-first quan trọng

Trong multi-repo, contract là điểm nối giữa:

- frontend
- API
- backend/job
- test/integration

Nếu không freeze contract sớm, mỗi repo sẽ tự giả định khác nhau.

---

## 16.2 Contract cần define gì?

Tùy feature, có thể gồm:

- API endpoint
- request body
- response body
- error code
- event payload
- job payload
- database-visible state
- status enum
- permission rule
- compatibility note

---

## 16.3 Contract change rule

Sau khi contract đã `frozen`, mọi thay đổi phải:

1. update `04.contracts.md`
2. update task packets liên quan
3. update tests
4. update status.md
5. notify repo owners

---

## 16.4 Contract ownership

| Contract type | Owner chính |
|---|---|
| API contract | tv-api owner + spec owner |
| UI usage contract | tv-app owner |
| Job/event contract | tv-api + tv-backend owners |
| Data contract | backend/API owner tùy architecture |
| Integration contract | feature owner |

---

# 17. Review, QA, integration và release gate

## 17.1 Repo review checklist

Mỗi repo PR cần check:

- [ ] đúng scope task packet
- [ ] không sửa unrelated files
- [ ] không thay đổi contract ngoài ý muốn
- [ ] tests pass
- [ ] docs updated nếu behavior đổi
- [ ] error handling đủ
- [ ] permission/security được kiểm tra
- [ ] logging/observability nếu cần

---

## 17.2 Cross-repo review checklist

Feature owner check:

- [ ] tv-app gọi đúng API contract
- [ ] tv-api expose đúng response/error format
- [ ] tv-backend consume đúng event/job payload
- [ ] status enum thống nhất
- [ ] retry/failure behavior thống nhất
- [ ] integration tests cover happy path và failure path

---

## 17.3 Integration test gate

Trước khi release:

- [ ] deploy branch/build tương ứng của 3 repo
- [ ] run migration nếu có
- [ ] seed data nếu cần
- [ ] run integration tests
- [ ] verify logs/metrics
- [ ] verify rollback plan

---

## 17.4 Release plan

File `08.release-plan.md` nên có:

```md
# Release Plan

## Release order
1. tv-backend migration/job support
2. tv-api endpoints
3. tv-app UI enablement

## Feature flag
- flag name:
- default:
- rollout:

## Rollback
- tv-app:
- tv-api:
- tv-backend:

## Monitoring
- metrics:
- logs:
- alerts:

## Post-release validation
- ...
```

---

# 18. Maintenance, governance và chống drift

## 18.1 Drift là gì?

Drift xảy ra khi:

- central contract nói một kiểu, code làm kiểu khác
- task packet cũ nhưng implementation đã đổi
- repo docs không update sau khi behavior đổi
- released feature docs không phản ánh thực tế
- AI dùng docs cũ để implement feature mới

---

## 18.2 Cách chống drift

### Rule 1 — Code PR phải link SDD change ID

Ví dụ:

```txt
SDD-Change: import-csv-user-data
```

### Rule 2 — Contract change cần review riêng

Không merge contract change lẫn implementation lớn nếu chưa review.

### Rule 3 — Docs update là part của DoD

Nếu behavior đổi, PR chưa done nếu docs chưa update.

### Rule 4 — Released feature phải update current-state

Sau khi release, nếu feature tạo capability mới, update:

```txt
current-state/domains/
current-state/contracts/
indexes/
repo docs
```

### Rule 5 — Archive không delete

Giữ `released` hoặc `archived` để trace decision.

---

## 18.3 Ownership

| Artifact | Owner |
|---|---|
| central feature spec | feature owner |
| contracts | spec owner + repo owner |
| repo docs | repo owner |
| task packet | feature owner + repo owner |
| current-state domain index | domain owner |
| integration plan | integration owner |

---

# 19. Checklist triển khai ban đầu

## 19.1 Setup central repo/folder

- [ ] tạo `tv-sdd` hoặc dùng folder `aidlc-docs`
- [ ] tạo `changes/active`, `changes/released`, `changes/archived`
- [ ] tạo `indexes`
- [ ] tạo `current-state`
- [ ] tạo `templates`
- [ ] tạo `design-artifacts`
- [ ] tạo `operations`

---

## 19.2 Inventory docs

- [ ] liệt kê docs của `tv-app`
- [ ] liệt kê docs của `tv-api`
- [ ] liệt kê docs của `tv-backend`
- [ ] tạo `repo-doc-index.md`
- [ ] tạo `domain-index.md`
- [ ] tạo `repo-responsibility.md`

---

## 19.3 Setup AI rules

- [ ] thêm `AGENTS.md` hoặc `.cursor/rules` cho từng repo
- [ ] thêm `CLAUDE.md` cho từng repo
- [ ] thêm rule “plan before edit”
- [ ] thêm rule “do not change contract without approval”
- [ ] thêm rule “read affected docs only”

---

## 19.4 Pilot feature

- [ ] chọn 1 feature thật
- [ ] tạo change folder
- [ ] tạo affected-docs
- [ ] tạo contract
- [ ] tạo task packets
- [ ] assign member
- [ ] implement
- [ ] integration test
- [ ] postmortem workflow

---

# 20. Checklist cho từng feature

## 20.1 Intake

- [ ] problem statement rõ
- [ ] business goal rõ
- [ ] affected users rõ
- [ ] success criteria rõ
- [ ] owner rõ

---

## 20.2 Brownfield

- [ ] affected repos identified
- [ ] affected docs listed
- [ ] existing capability mapped
- [ ] reuse candidates listed
- [ ] risks/unknowns listed

---

## 20.3 Spec

- [ ] requirements written
- [ ] non-goals written
- [ ] edge cases written
- [ ] acceptance criteria written

---

## 20.4 Contract

- [ ] API/event/data contract drafted
- [ ] repo owners reviewed
- [ ] contract frozen
- [ ] tests planned

---

## 20.5 Tasks

- [ ] task packet for tv-app
- [ ] task packet for tv-api
- [ ] task packet for tv-backend
- [ ] owner assigned
- [ ] local DoD clear

---

## 20.6 Implementation

- [ ] local plans approved
- [ ] implementation done
- [ ] local tests pass
- [ ] docs updated
- [ ] PR reviewed

---

## 20.7 Integration

- [ ] deploy integration env
- [ ] run integration tests
- [ ] verify logs/metrics
- [ ] QA sign-off
- [ ] release plan ready

---

## 20.8 Release

- [ ] release completed
- [ ] post-release validation passed
- [ ] current-state updated
- [ ] change moved to released

---

# 21. Prompt templates cho Cursor/Claude

## 21.1 Brownfield discovery prompt

```txt
You are helping with brownfield discovery for a multi-repo SDD/AIDLC workflow.

Goal:
Identify only the existing docs and code areas relevant to this feature.

Feature:
<feature summary>

Repos:
- tv-app
- tv-api
- tv-backend

Instructions:
1. Do not read all docs.
2. Start from repo-doc-index.md and domain-index.md.
3. Identify affected domains.
4. List docs to read first.
5. List docs to read only if needed.
6. List likely code areas.
7. List unknowns and risks.
8. Do not propose implementation yet.

Output:
- affected repos
- affected domains
- affected docs
- existing capabilities
- gaps
- risks
- open questions
```

---

## 21.2 Generate feature spec prompt

```txt
Create a central SDD feature spec.

Input:
- feature request
- brownfield discovery
- affected docs
- known constraints

Rules:
- Do not duplicate repo docs.
- Focus on change/delta.
- Define business requirements.
- Define non-goals.
- Define acceptance criteria.
- Identify contracts that must be created or changed.
- Keep implementation detail out unless required for integration.

Output files:
- 00.overview.md
- 01.requirements.md
- 04.contracts.md draft
- 07.status.md initial
```

---

## 21.3 Generate task packets prompt

```txt
Generate repo task packets from this central feature spec.

Rules:
- One task packet per repo.
- Each packet must be self-contained enough for the repo owner.
- Do not copy full global spec.
- Include link/reference to source of truth.
- Include scope and out-of-scope.
- Include affected local docs.
- Include relevant contracts.
- Include tests and definition of done.
- Include "ask before expanding scope".

Repos:
- tv-app
- tv-api
- tv-backend
```

---

## 21.4 Repo implementation planning prompt

```txt
Read the repo task packet and affected local docs only.

Do not edit files yet.

Produce:
1. implementation plan
2. affected files/modules
3. test plan
4. risks
5. assumptions
6. questions/blockers

Rules:
- Stay inside this repo's scope.
- Do not change contract unless explicitly requested.
- Do not refactor unrelated code.
- Prefer smallest safe change.
```

---

## 21.5 Repo implementation prompt

```txt
Implement the approved plan.

Rules:
- Follow task packet scope.
- Keep changes minimal.
- Add/update tests.
- Update repo docs if behavior changes.
- Do not modify unrelated modules.
- Stop and ask if contract mismatch is found.

After implementation, output:
1. files changed
2. tests added/updated
3. tests run
4. contract compliance summary
5. docs updated
6. remaining risks
```

---

## 21.6 Review prompt

```txt
Review this repo change against the SDD task packet.

Check:
- scope compliance
- contract compliance
- tests
- docs
- error handling
- security/permission
- backward compatibility
- unintended side effects

Output:
- approve/block
- issues by severity
- required fixes
- optional improvements
```

---

## 21.7 Integration review prompt

```txt
Review cross-repo integration for this feature.

Inputs:
- central overview
- contracts
- task packets
- PR summaries from tv-app, tv-api, tv-backend
- integration test plan

Check:
- contract consistency
- status enum consistency
- error handling consistency
- missing integration tests
- release order risks
- rollback risks

Output:
- integration readiness
- blockers
- required tests
- release notes
```

---

## 21.8 Post-release update prompt

```txt
Feature has been released.

Update SDD/current-state indexes.

Inputs:
- released feature spec
- final PR summaries
- final contracts
- changed repo docs

Tasks:
1. Update current-state domain docs if new capability exists.
2. Update contract index.
3. Update feature index.
4. Move change from active to released.
5. Create release summary.
6. List any follow-up docs debt.
```

---

# 22. Rủi ro thường gặp và cách xử lý

## 22.1 Rủi ro: tv-sdd trở thành nơi duplicate toàn bộ docs

Dấu hiệu:

- feature spec quá dài
- copy nhiều đoạn từ repo docs
- update một behavior phải sửa 3-4 file

Cách xử lý:

- feature spec chỉ ghi delta
- repo docs là source chi tiết
- central chỉ index/reference

---

## 22.2 Rủi ro: AI đọc quá nhiều context

Dấu hiệu:

- prompt rất chậm
- câu trả lời lan man
- AI sửa unrelated files
- token usage cao

Cách xử lý:

- dùng affected-docs
- chia context theo L0-L5
- yêu cầu AI plan trước
- yêu cầu “ask before expanding scope”

---

## 22.3 Rủi ro: contract drift

Dấu hiệu:

- tv-app expect field khác tv-api trả về
- tv-backend event payload thiếu field
- status enum không thống nhất

Cách xử lý:

- contract-first
- freeze contract
- contract test
- status.md track contract change
- notify repo owners khi contract đổi

---

## 22.4 Rủi ro: mỗi repo tự diễn giải requirement khác nhau

Dấu hiệu:

- implementation lệch nhau
- mỗi repo tự thêm behavior
- UI/API/job không khớp

Cách xử lý:

- global requirements là source of truth
- task packet chỉ là implementation slice
- feature owner review task packets
- integration owner review PR summaries

---

## 22.5 Rủi ro: docs không được update sau release

Dấu hiệu:

- feature sau dùng docs cũ
- AI đưa ra plan sai
- member mới bị hiểu nhầm

Cách xử lý:

- docs update là release gate
- post-release update prompt
- current-state update checklist
- periodic docs audit

---

## 22.6 Rủi ro: AI tự refactor quá rộng

Dấu hiệu:

- PR lớn bất thường
- nhiều unrelated files
- khó review
- bug regression

Cách xử lý:

- task packet có Out of scope
- prompt “smallest safe change”
- plan-before-edit
- review scope compliance

---

# 23. Kết luận

Mô hình nên triển khai là:

```txt
Existing repo docs
  = detailed local source of truth

tv-sdd / aidlc-docs
  = central orchestration, index, contract, feature delta, integration

Repo task packets
  = implementation slice cho từng member/repo

Cursor/Claude
  = executor trong boundary rõ ràng
```

Không nên bắt đầu bằng việc viết lại toàn bộ spec hệ thống. Dự án đã chạy rồi, nên bắt đầu bằng:

```txt
1. inventory docs hiện có
2. tạo central index
3. tạo SDD skeleton
4. chọn một feature pilot
5. chạy full lifecycle
6. chuẩn hóa template
7. automation sau
```

Điểm quan trọng nhất để workflow này thành công:

- contract-first
- task packet nhỏ
- affected-docs rõ
- repo ownership rõ
- AI context loading có kiểm soát
- docs update là part của Definition of Done
- integration gate trước release

---

# Appendix A — Minimal files cần tạo ngay

Nếu muốn bắt đầu trong 1 ngày, chỉ cần tạo các file sau:

```txt
tv-sdd/aidlc-docs/
  README.md
  indexes/
    repo-doc-index.md
    domain-index.md
  current-state/
    architecture/
      repo-responsibility.md
  changes/
    active/
    released/
    archived/
  templates/
    feature-overview.template.md
    affected-docs.template.md
    contracts.template.md
    task-packet.template.md
    status.template.md
```

Với mỗi repo:

```txt
<repo>/
  AGENTS.md
  CLAUDE.md
  aidlc-docs/
    README.md
    changes/
      active/
      released/
      archived/
```

---

# Appendix B — Naming convention

## Change ID

Dùng kebab-case:

```txt
import-csv-user-data
admin-user-bulk-update
notification-retry-policy
```

## File prefix

Dùng số thứ tự để đọc theo flow:

```txt
00.overview.md
01.requirements.md
02.brownfield-discovery.md
03.affected-docs.md
04.contracts.md
05.implementation-plan.md
06.integration-test-plan.md
07.status.md
08.release-plan.md
```

## Task packet

```txt
task-packets/tv-app.md
task-packets/tv-api.md
task-packets/tv-backend.md
```

---

# Appendix C — README mẫu cho `aidlc-docs`

```md
# AIDLC Docs

This folder manages SDD/AIDLC workflow for this repository/project.

## Main concepts

- `changes/active`: ongoing changes
- `changes/released`: released changes
- `changes/archived`: old changes
- `current-state`: current system indexes and architecture
- `templates`: reusable templates
- `indexes`: maps to existing repo docs

## Rules

1. Do not duplicate repo docs into central SDD.
2. Use affected-docs to control context.
3. Contract changes must be reviewed.
4. Repo task packets must define scope and out-of-scope.
5. Update docs when behavior changes.
```

---

# Appendix D — Repo responsibility template

```md
# Repo Responsibility

## tv-app

Owns:
- UI
- frontend routing
- client-side validation
- API client usage
- user interaction behavior

Does not own:
- API schema
- backend job behavior
- database migration

## tv-api

Owns:
- public/admin API
- request validation
- permission enforcement
- response format
- enqueueing backend jobs/events

Does not own:
- frontend UI
- long-running worker internals unless explicitly part of API service

## tv-backend

Owns:
- batch jobs
- async processing
- notifications
- data processing
- retries/idempotency

Does not own:
- frontend UI
- public API response shape unless shared contract requires it
```

---

# Appendix E — Contract index template

```md
# Contract Index

## User APIs

| Contract | Provider | Consumers | Current doc |
|---|---|---|---|
| GET /users | tv-api | tv-app | tv-api/docs/... |
| GET /users/{id} | tv-api | tv-app | tv-api/docs/... |
| POST /users | tv-api | tv-app | tv-api/docs/... |

## Jobs/Events

| Contract | Producer | Consumer | Current doc |
|---|---|---|---|
| user-import-created | tv-api | tv-backend | tv-sdd/design-artifacts/events/... |
```

---

# Appendix F — Integration status board template

```md
# Integration Status Board

## Feature
<feature-id>

## Contract status
draft | frozen | changed-after-freeze

## Repo readiness

| Repo | Branch | PR | Tests | Docs | Ready |
|---|---|---|---|---|---|
| tv-app |  |  |  |  | no |
| tv-api |  |  |  |  | no |
| tv-backend |  |  |  |  | no |

## Integration environment
- URL:
- deploy version:
- seed data:

## Test status
- [ ] happy path
- [ ] invalid input
- [ ] permission denied
- [ ] partial failure
- [ ] retry/rollback
```

---

# Appendix G — Recommended first 5 tasks

1. Tạo `repo-doc-index.md` từ docs hiện có.
2. Tạo `repo-responsibility.md`.
3. Tạo templates trong `aidlc-docs/templates`.
4. Tạo `AGENTS.md` và `CLAUDE.md` tối giản cho từng repo.
5. Chạy pilot feature `import-csv-user-data` theo lifecycle đầy đủ.

---

# Appendix H — Tài liệu tham khảo

- Cursor Rules documentation
- Claude Code memory / CLAUDE.md documentation

Lưu ý: Các file rule/memory của Cursor và Claude có thể thay đổi theo phiên bản tool. Khi triển khai chính thức, nên kiểm tra lại documentation chính thức của tool đang dùng.
