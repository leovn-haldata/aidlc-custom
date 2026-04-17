---
description: >
  AI agent workflow optimization. Parallel task execution, context window management,
  build troubleshooting, and target cycle times for efficient AI-assisted development.
  This rule covers AI agent efficiency, NOT application runtime performance.
  Reference this when optimizing AI workflows or troubleshooting build issues.
---

# AI Workflow Optimization

> **Scope**: This rule governs how the AI agent works efficiently (parallel execution, context management, cycle times). For **application runtime performance** (response time, memory, scaling), refer to the project's per-folder rules or the design/test artifacts.

## Parallel Task Execution

Independent operations should always run in parallel:

```markdown
# GOOD: Parallel execution
Run 3 independent expert reviews concurrently:
1. Security Reviewer: Analyze auth.ts
2. Code Reviewer: Review cache module
3. TDD Expert: Check test coverage

# BAD: Unnecessary sequential execution
First security, then code review, then TDD -- when they are independent.
```

## Context Management

For large tasks that strain the context window:
- Split large refactoring into focused, independent sub-tasks
- Process one module/unit at a time
- Use Plan Mode for complex multi-file operations
- Reference artifact files instead of inlining their full content

## Build Troubleshooting

When a build fails:
1. Run `/aidlc-fix` or activate the Build Error Resolver role
2. Read error messages carefully -- fix the root cause, not symptoms
3. Apply fixes incrementally, verify after each fix
4. If fixes exceed 20 lines, pause and present the plan

## Token Cost Tracking

Track AI token usage to identify optimization opportunities:

### What to Track

| Metric | How | Frequency |
|--------|-----|-----------|
| Tokens per command execution | Note approximate token count after each `/aidlc-*` command | Per command |
| Total tokens per change (intake → release) | Sum across all commands for a change | Per change |
| Token cost by spec mode | Compare average cost across micro / light / feature / full | Per sprint |

### Optimization Strategies

- **Scope references, don't inline**: Point to artifact files instead of pasting their full content
- **Unit-scoped context**: When working on a specific unit/module, reference only that unit's artifacts
- **Incremental loading**: Load spec artifacts only when the current step needs them
- **Prune stale context**: Long conversations accumulate noise; start fresh sessions for new commands
- **Review rule loading**: Check that only relevant rules are loaded per interaction (see `docs/adoption-guide.md` Token Optimization section)

### When to Act

- If a single command consistently uses > 50% of the context window → split the task or reduce referenced artifacts
- If token cost per sprint is trending up without matching productivity gains → review at retrospective
- Track findings in the sprint retrospective under "Process Observations"

## Efficient Workflow

- For light-spec tasks: aim for intake-to-review in under 2 hours
- For feature-spec tasks: aim for intake-to-review in under 1 day
- Skip optional steps when they add no value (but never skip confirmation and review gates)
- Batch related light-spec changes when possible
