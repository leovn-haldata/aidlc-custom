---
description: >
  Coding style guidelines. Base rules that apply to all code, plus per-folder rule delegation.
  When working in a specific code folder, ALWAYS check for and apply that folder's own rules.
globs:
  - "**/*.ts"
  - "**/*.tsx"
  - "**/*.js"
  - "**/*.jsx"
  - "**/*.py"
  - "**/*.go"
  - "**/*.java"
  - "**/*.rb"
  - "**/*.rs"
  - "**/*.php"
---

# Coding Style

## Per-Folder Rules (Priority)

**IMPORTANT**: Each code folder (e.g., `backend/`, `frontend/`, `ai/`, `deployment/`) may contain its own `rules/` directory with folder-specific coding rules.

**When editing code in any folder, you MUST:**

1. Check if the folder has a `rules/` directory (e.g., `backend/rules/`, `frontend/rules/`)
2. If rules exist, read and apply them -- they take priority over these base rules for that folder
3. If no folder-specific rules exist, apply only the base rules below

```
project/
├── backend/
│   ├── rules/           ← Apply these when working in backend/
│   │   ├── python.md
│   │   └── fastapi.md
│   └── src/
├── frontend/
│   ├── rules/           ← Apply these when working in frontend/
│   │   ├── typescript.md
│   │   └── react.md
│   └── src/
├── deployment/
│   ├── rules/           ← Apply these when working in deployment/
│   │   └── docker.md
│   └── ...
└── shared/
    └── rules/           ← Apply these when working in shared/
```

This pattern keeps rules close to the code they govern, making them easier to maintain and update per tech stack.

---

## Base Rules (Apply to ALL code when no folder-specific rule overrides)

> **Note**: These base rules describe language-agnostic **principles**. For language-specific
> patterns and code examples, see the per-folder rules (`backend/rules/`, `frontend/rules/`, etc.).

### Immutability

Prefer creating new objects over mutating existing ones. Each language has its own patterns:

- **JavaScript/TypeScript**: Spread operator (`{ ...obj, key: value }`), `Object.freeze()`, Immer
- **Python**: `dataclasses(frozen=True)`, dict unpacking (`{**d, 'key': value}`), named tuples
- **Go**: Return new structs instead of modifying pointer receivers
- **Java/Kotlin**: Use `final` / `val`, return new instances from methods
- **Rust**: Default immutability (`let` vs `let mut`)

### File Organization

**Many small files > few large files**:
- High cohesion, low coupling
- Standard: 200-400 lines. Hard limit: 800 lines.
- Extract utilities when components grow large.
- Organize by feature/domain, not by type.

### Error Handling

Implement comprehensive error handling at every layer:

- Catch exceptions at appropriate levels (not too broad, not too narrow)
- Log errors with context (operation name, relevant IDs, stack trace)
- Throw domain-specific errors, not raw language exceptions
- Never swallow errors silently
- Return user-friendly error messages (no internal details in API responses)

### Input Validation

Always validate user input on the server side:

- Use schema-based validation (Zod/TypeScript, Pydantic/Python, validator/Go, Bean Validation/Java)
- Validate at the API boundary before business logic
- Client-side validation is for UX only -- never trust it for security
- Check file uploads: MIME type, size, extension, encoding
- Use parameterized queries for ALL database operations

### Naming Conventions

- Variables/functions: descriptive, action-oriented (`getUserById`, not `data`)
- Booleans: prefix with `is`, `has`, `can`, `should`
- Constants: UPPER_SNAKE_CASE
- Classes/Types: PascalCase
- Files: match the primary export name

### Code Quality Checklist

Verify before completing any edit:
- [ ] Code is readable with clear naming
- [ ] Functions are small (< 50 lines)
- [ ] Files are focused (< 800 lines)
- [ ] No deep nesting (> 4 levels)
- [ ] Proper error handling at every layer
- [ ] No console.log / print statements left in production code
- [ ] No hardcoded values
- [ ] Immutability patterns used throughout
- [ ] Per-folder rules checked and applied (if they exist)
