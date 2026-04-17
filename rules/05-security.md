---
description: >
  Security guidelines. No hardcoded secrets, input validation, SQL injection prevention,
  XSS prevention, PII handling, authentication and authorization requirements.
  Applied when working with code files.
globs:
  - "**/*.ts"
  - "**/*.tsx"
  - "**/*.js"
  - "**/*.jsx"
  - "**/*.py"
  - "**/*.go"
  - "**/*.php"
  - "**/*.yaml"
  - "**/*.yml"
  - "**/*.json"
  - "**/*.toml"
  - "**/*.env.example"
  - "**/Dockerfile"
  - "**/docker-compose*.yml"
---

# Security Guidelines

## Pre-Commit Security Checklist

- [ ] No hardcoded secrets (API keys, passwords, tokens, DB credentials)
- [ ] All user input validated server-side
- [ ] SQL injection prevented (parameterized queries only)
- [ ] XSS prevented (HTML output sanitized/escaped)
- [ ] CSRF protection enabled on state-changing endpoints
- [ ] Authentication verified on all protected endpoints
- [ ] Authorization enforced (tenant isolation, role-based access)
- [ ] Rate limiting on sensitive endpoints (auth, file upload)
- [ ] Error messages do not leak internal details
- [ ] PII is not logged or exposed in responses

## Secrets Management

```typescript
// NEVER do this
const apiKey = "sk-proj-xxxxx"

// ALWAYS use environment variables
const apiKey = process.env.API_KEY
if (!apiKey) {
  throw new Error('API_KEY not configured')
}
```

- `.env` files MUST be in `.gitignore`
- Use platform-native credential mechanisms (ADC, IAM roles) in production
- Never log secret values

## PII Handling

- Mask PII before persistent storage or display
- Never log personal names, addresses, or raw text that may contain personal info
- Error responses must not leak personal data
- Audit data access patterns for PII-containing tables

## Input Validation

- Validate ALL user input server-side (client-side validation is for UX only)
- Check file uploads: MIME type, size, extension, encoding
- Use parameterized queries for ALL database operations
- Sanitize output to prevent XSS
- Use strict comparison (`===`) to avoid type juggling

## When a Security Issue is Found

1. **STOP immediately** -- do not continue implementation
2. Assess severity: CRITICAL / HIGH / MEDIUM / LOW
3. Fix CRITICAL issues before proceeding
4. Rotate any leaked credentials immediately
5. Review the codebase for similar issues
6. Document the finding and fix
