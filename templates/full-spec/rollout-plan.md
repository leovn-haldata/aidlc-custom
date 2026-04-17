# Rollout Plan: <change-id>

## Change Reference
- **Design**: [design.md](./design.md)
- **Mode**: full-spec

## Rollout Strategy

| Phase | Scope | Duration | Success Gate |
|-------|-------|----------|-------------|
| Phase 1 | Internal / staging | _N days_ | _All smoke tests pass_ |
| Phase 2 | Canary (N% traffic) | _N days_ | _Error rate < threshold_ |
| Phase 3 | Full rollout | _N days_ | _All metrics normal_ |

## Feature Flags

| Flag Name | Default | Description |
|-----------|---------|-------------|
| `feature_<change-id>_enabled` | false | Master toggle for this feature |

## Pre-Rollout Checklist

- [ ] All tests pass on staging
- [ ] Database migration applied and verified
- [ ] API contracts validated
- [ ] Monitoring dashboards configured
- [ ] Alerting rules set up
- [ ] Rollback procedure tested
- [ ] Communication sent to stakeholders

## Rollback Procedure

1. Disable feature flag: `feature_<change-id>_enabled = false`
2. If DB migration involved: run down migration `<change-id>_down`
3. If API contract changed: revert to previous version
4. Verify rollback: run smoke tests
5. Notify stakeholders of rollback

**Estimated rollback time**: _N minutes_

## Monitoring Checkpoints

| Metric | Normal Range | Alert Threshold | Dashboard |
|--------|-------------|----------------|-----------|
| Error rate | < 0.1% | > 1% | _link_ |
| Response time (p95) | < 200ms | > 500ms | _link_ |
| _Custom metric_ | _range_ | _threshold_ | _link_ |

## Communication Plan

| When | Who | Channel | Message |
|------|-----|---------|---------|
| Before rollout | _Stakeholders_ | _Email/Slack_ | _Feature going live_ |
| After Phase 3 | _Team_ | _Slack_ | _Rollout complete_ |
| If rollback | _All_ | _Slack + Email_ | _Issue detected, rolling back_ |
