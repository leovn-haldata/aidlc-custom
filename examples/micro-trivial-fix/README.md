# Example: micro -- Fix Typo in Error Message

## Scenario

The login error message says "Pasword incorrect" instead of "Password incorrect". This is a trivial 1-line fix with zero risk.

## Why micro (not light-spec)?

- Single character fix in a single file
- No behavior change (just a string correction)
- Root cause is obvious (typo)
- No impact tracing needed
- Takes less than 5 minutes

## Flow Used

```
1. Developer spots the typo
2. Fix the string in the source file
3. Run existing tests (should still pass)
4. Commit with descriptive message:
   "fix: correct typo in login error message ('Pasword' → 'Password')"
5. Push to develop (or quick PR if team policy requires)
```

No spec artifacts generated. No `/aidlc-intake`, no `/aidlc-spec`, no `/aidlc-confirm`.

## Artifacts Produced

None -- commit message is the only documentation.

## Branch: `fix/typo-login-error` or direct commit on develop

## Time: ~5 minutes total

## When This Becomes light-spec

If the "typo" turns out to be in multiple places, or if fixing it reveals that the error handling logic itself is wrong, escalate to light-spec:

```
"I started as micro but discovered the error message is generated from a shared utility
that affects 5 other endpoints. Escalating to light-spec to trace impact."
```
