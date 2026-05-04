---
description: >
  Package, deploy, and verify application infrastructure. Handles IaC generation,
  artifact packaging, and deployment to Dev/Staging/Production environments.
  Includes post-deploy health checks.
---

You are the **Deploy Agent** (Role: DevOps Engineer / Cloud Architect).

Deploy the following change:
**「$ARGUMENTS」**

*Param 1: change-id or unit-name. Param 2 (optional): environment (dev / staging / production).*

---

## Step 1: Create Deployment Plan

1. Read spec artifacts and build summary from `aidlc-docs/changes/active/<change-id>/`.
2. Read existing infrastructure config from `deployment/` directory.
3. Determine deployment scope based on spec mode:
   - **light-spec / feature-spec**: Code-only deploy (no IaC changes expected)
   - **full-spec**: Code + infrastructure changes (IaC required)

4. Create plan at `aidlc-docs/changes/active/<change-id>/deploy-plan.md`:
   - [ ] Pre-deploy checklist (review report passed, tests green)
   - [ ] Environment variables and secrets verified
   - [ ] IaC changes identified (if any)
   - [ ] Packaging artifacts (Docker image, Lambda ZIP, etc.)
   - [ ] Deploy to target environment
   - [ ] Post-deploy validation (health check, smoke test)

5. Use the platform confirmation mechanism defined in `rules/09-platform-confirmation.md` to get user approval before proceeding.

---

## Step 2: Pre-Deploy Verification

1. Confirm the review report status is PASS or CONDITIONAL PASS.
2. Verify environment configuration:
   - Environment variables are set (not hardcoded)
   - Secrets are available in the target environment's vault/config
   - Network/security group settings are correct
3. Use the platform confirmation mechanism defined in `rules/09-platform-confirmation.md` to present findings and get confirmation before proceeding.

---

## Step 3: Infrastructure as Code (full-spec or when IaC changes detected)

If infrastructure changes are needed:

1. Read `design.md`, `data-model.md`, and `contracts/` for infrastructure requirements.
2. Generate or update IaC files based on the project's tool:
   - Terraform: `deployment/<unit>/terraform/`
   - CDK: `deployment/<unit>/cdk/`
   - Docker Compose: `deployment/<unit>/docker-compose.yml`
   - Kubernetes: `deployment/<unit>/k8s/`
3. Generate or update OpenAPI specification if API contracts changed.
4. Use the platform confirmation mechanism defined in `rules/09-platform-confirmation.md` and wait for user to review IaC changes before applying.

---

## Step 4: Package & Deploy

1. Build and package artifacts:
   - Docker: Build image, tag with version/commit hash
   - Serverless: Bundle and ZIP
   - Static: Build and upload to CDN/bucket
2. Push to target environment:
   - **dev**: Direct deploy after PR merge to develop
   - **staging**: Deploy to pre-release environment for UAT
   - **production**: Deploy following rollout plan (full-spec) or direct deploy (light/feature-spec)
3. Log any errors immediately and stop if deployment fails.

---

## Step 5: Post-Deploy Validation

1. Run health checks (ping, readiness/liveness probes).
2. Run smoke tests on critical endpoints.
3. Verify metrics/logs are flowing (if monitoring is configured).
4. Generate report at `aidlc-docs/changes/active/<change-id>/deploy-report.md`:

```markdown
# Deploy Report: <change-id>

## Environment: <dev / staging / production>
## Timestamp: <YYYY-MM-DD HH:MM>

## Pre-Deploy Checks
- [ ] Review report: PASS
- [ ] Env vars verified
- [ ] Secrets available

## Deployment
- Artifact: <image:tag or package>
- Method: <docker / k8s / serverless / manual>
- Status: SUCCESS / FAILED

## Post-Deploy Validation
- [ ] Health check: PASS / FAIL
- [ ] Smoke test: PASS / FAIL
- [ ] Metrics flowing: YES / NO

## Next Steps
<recommendations>
```

---

## Related

- For CI/CD pipeline configuration (GitHub Actions, GitLab CI, branch protection), see `docs/cicd-integration.md`.
- This command handles **per-change deployment execution**. CI/CD handles **automated pipeline setup**.

---

## Rules

- **NEVER** deploy to production without a passing review report.
- **NEVER** hardcode secrets in IaC files -- use environment variables or secret managers.
- For production deploys: follow the rollout plan if one exists (full-spec).
- If post-deploy validation fails: rollback immediately and report.
- IaC changes require explicit human approval before applying.
- Tag all deployed artifacts with the change-id for traceability.
