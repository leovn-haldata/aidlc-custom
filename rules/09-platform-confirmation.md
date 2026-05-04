---
description: >
  Defines how to request human confirmation depending on the AI platform in use.
  All commands reference this rule instead of hardcoding platform-specific tool calls.
---

# Platform Confirmation Tool

When a command reaches a gate that requires human confirmation before proceeding,
use the appropriate mechanism for the current platform:

| Platform | Mechanism |
|---|---|
| **Cursor** | `AskQuestion` tool with a title, prompt, and labeled options |
| **Claude Code** | Call `EnterPlanMode` before presenting, then `ExitPlanMode` after |
| **Continue.dev / Other** | Present findings in bold and stop — wait for user reply in chat |

---

## Cursor — AskQuestion pattern

```
AskQuestion({
  title: "<Gate Name>",
  questions: [{
    id: "decision",
    prompt: "<summary of what you are presenting>\n\nHow would you like to proceed?",
    options: [
      { id: "confirm", label: "Confirm — proceed" },
      { id: "revise",  label: "Revise — go back and adjust" },
      { id: "reject",  label: "Reject — cancel this change" }
    ]
  }]
})
```

Adapt the options to the specific gate (e.g. mode selection offers override choices).

---

## Claude Code — Plan Mode pattern

```
1. Call EnterPlanMode tool.
2. Present your findings or plan.
3. Call ExitPlanMode tool to request approval.
```

---

## Continue.dev / Other — Text pattern

```
**[Gate: <Gate Name>]**
<present findings>

Please reply: confirm / revise / reject
```

---

## Rule

Every command that has a human-in-the-loop gate MUST use this rule.
Never hardcode a platform-specific tool call directly in a command file.
