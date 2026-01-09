---
name: code-review
description: Review recent work for best practices, efficiency, and security. Use after completing implementation to verify no broken references, security issues, or regressions. Triggers on "review my code", "check my work", "verify implementation", or when asked to audit recent changes.
user_invocable: true
---

# Code Review

Systematically review recent implementation work for quality, security, and correctness.

## Review Process

### Step 1: Identify Recent Changes

First, check git for modifications:

```bash
git diff HEAD~3 --name-only
git log --oneline -5
```

Focus on recently modified files, especially:
- Controllers (authorization, validation)
- Views/templates (XSS, CSRF)
- JavaScript (DOM manipulation, API calls)
- Routes (middleware)

### Step 2: Security Checklist

**PHP/Laravel:**
- [ ] Input validation with proper rules and max lengths
- [ ] Authorization checks (ownership verification, `canBeViewedBy()`)
- [ ] CSRF protection on forms (`@csrf`)
- [ ] No raw SQL queries (use Eloquent/Query Builder)
- [ ] Sensitive data not logged (passwords, tokens, API keys)
- [ ] Mass assignment protection (fillable/guarded)

**JavaScript:**
- [ ] No `innerHTML` with unsanitized user input
- [ ] API calls include CSRF token in headers
- [ ] No exposed secrets or API keys
- [ ] Error handling doesn't leak sensitive info

**Blade Templates:**
- [ ] Use `{{ }}` for output (auto-escapes)
- [ ] Use `{!! !!}` only for trusted HTML
- [ ] Forms have `@csrf` directive

### Step 3: Best Practices Checklist

**Code Quality:**
- [ ] No dead code or unused imports
- [ ] No broken function/method references
- [ ] Consistent naming conventions
- [ ] Proper error handling with user-friendly messages
- [ ] No console.log in production code

**Efficiency:**
- [ ] No N+1 query problems (use eager loading)
- [ ] Appropriate caching where beneficial
- [ ] No unnecessary database calls
- [ ] Reasonable AJAX payload sizes

**Laravel Specific:**
- [ ] Form requests for complex validation
- [ ] Route model binding where appropriate
- [ ] Service layer for business logic
- [ ] Jobs for background processing

### Step 4: Functional Verification

- [ ] All referenced functions/methods exist
- [ ] All routes properly defined
- [ ] All required views exist
- [ ] JavaScript event handlers properly bound
- [ ] No typos in method/function names

### Step 5: Report Format

Provide summary table:

| Category | Status | Issues Found |
|----------|--------|--------------|
| Security | Pass/Fail | Description |
| Best Practices | Pass/Fail | Description |
| Efficiency | Pass/Fail | Description |
| Broken References | Pass/Fail | Description |

Then categorize issues:

1. **Critical** - Must fix before deploy
2. **Warnings** - Should fix soon
3. **Suggestions** - Nice to have

## Example Review

```markdown
## Code Review: Dashboard SPA

### Files Reviewed
- public/js/dashboard-spa.js
- resources/views/partials/dashboard/*.blade.php
- app/Http/Controllers/StoryController.php

### Summary

| Category | Status | Issues |
|----------|--------|--------|
| Security | Pass | CSRF tokens present, proper validation |
| Best Practices | Pass | Clean code, proper error handling |
| Efficiency | Pass | No N+1 issues detected |
| Broken References | Fail | 1 dead function call |

### Critical Issues
1. **generate-story.blade.php:45** - Calls `openProfilePanel()` which was removed

### Warnings
None

### Suggestions
- Remove unused CSS for profile panel
```

## When to Run

- After completing any implementation work
- Before creating pull requests
- After refactoring code
- When debugging unexpected behavior
