# Estimation Guide

## Estimation Methods

### T-Shirt Sizing (Quick, for Intake)

Use during `/aidlc-intake` for initial classification:

| Size | Effort | Duration | Spec Mode |
|------|--------|----------|-----------|
| **XS** | < 1 hour | Same day | micro |
| **S** | 1h - 2 days | 1-2 days | light-spec |
| **M** | 2 - 5 days | 3-5 days | feature-spec |
| **L** | 1 - 2 weeks | 5-10 days | feature-spec |
| **XL** | 2+ weeks | 10+ days | full-spec |

### Story Points (for Sprint Planning)

Use a modified Fibonacci scale: **1, 2, 3, 5, 8, 13**

| Points | Meaning | Complexity | Uncertainty |
|--------|---------|-----------|-------------|
| **1** | Trivial, well-understood | Very Low | None |
| **2** | Simple, clear solution | Low | Minimal |
| **3** | Moderate, some decisions needed | Medium | Some |
| **5** | Complex, multiple components | High | Moderate |
| **8** | Very complex, cross-cutting | Very High | Significant |
| **13** | Extremely complex, high risk | Extreme | High |

**Rule**: If an item is estimated at **13+**, break it down into smaller items before committing to a sprint.

### Hour-Based Estimation (for Tasks)

Use in `/aidlc-tasks` for individual task estimates:

| Estimate | Guideline |
|----------|-----------|
| 30 min | Config change, simple fix |
| 1-2 hours | Single function implementation + test |
| 3-4 hours | Small module with tests |
| 6-8 hours (1 day) | Complex module or integration |
| **> 8 hours** | Break into smaller tasks |

**Rule**: No single task should exceed 8 hours. If it does, decompose further.

## Estimation Process

### At Intake (`/aidlc-intake`)

1. PM/BrSE provides requirement description
2. AI classifies using the decision matrix (LOC, files, duration, team size)
3. Apply T-shirt size for quick sanity check
4. Recommend spec mode
5. Developer validates or overrides

### At Sprint Planning

1. Each item already has a spec mode from intake
2. Team estimates in story points using Planning Poker:
   - Everyone picks a card simultaneously
   - If estimates differ by > 2 steps (e.g., 3 vs 8), discuss and re-vote
   - Use the average or the higher estimate for risk-averse planning
3. Sprint capacity = average team velocity (last 3 sprints)
4. Reserve 15-20% for unplanned work

### At Task Breakdown (`/aidlc-tasks`)

1. Each task gets an hour estimate
2. Total hours should roughly match the story point allocation
3. If total hours exceed sprint capacity, reduce scope or split across sprints

## Velocity Tracking

### How to Track

```markdown
## Velocity History

| Sprint | Planned (pts) | Completed (pts) | Velocity |
|--------|--------------|----------------|----------|
| S01 | 24 | 21 | 21 |
| S02 | 24 | 26 | 26 |
| S03 | 24 | 22 | 22 |
| **Average** | | | **23** |
```

### Using Velocity

- **Plan next sprint** based on average velocity (not best-case)
- **Trend matters**: 3 sprints declining → discuss at retro
- **New team**: Start with conservative estimates, adjust after 2-3 sprints
- **Velocity is NOT a performance metric** -- it's a planning tool

## Common Estimation Pitfalls

| Pitfall | Solution |
|---------|----------|
| Optimistic bias | Add 20-30% buffer for unknowns |
| Ignoring test time | TDD takes ~40% of total dev time -- include it |
| Ignoring review time | PR review + fix cycles add 10-20% |
| "It's easy" fallacy | If you haven't done it before, it's not easy |
| Scope creep | Estimate against the defined scope in `proposal.md`, not imagined extras |
| Anchoring | Don't let the first estimate bias others (use Planning Poker) |

## Estimation Refinement

Improve estimates over time:

1. **After each sprint**: Compare estimated vs actual for completed items
2. **Track accuracy**: Are we consistently over or under-estimating?
3. **Adjust multiplier**: If actuals are 1.5x estimates, apply 1.5x going forward
4. **Per-category patterns**: Track which types of work we estimate poorly

```markdown
## Estimation Accuracy Log

| Change ID | Estimated | Actual | Ratio | Notes |
|-----------|----------|--------|-------|-------|
| CHG-015 | 5 pts | 5 pts | 1.0 | Accurate |
| CHG-016 | 3 pts | 5 pts | 1.67 | Underestimated DB impact |
| CHG-017 | 1 pt | 1 pt | 1.0 | Accurate |
```
