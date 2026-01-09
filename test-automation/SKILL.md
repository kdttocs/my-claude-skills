---
name: test-automation
description: Run comprehensive automated tests and code reviews. Supports two modes - (1) Infrastructure/Health Review for server, database, API, and system checks via SSH, and (2) Code Review for security, best practices, and quality analysis of recent changes. When invoked ambiguously, asks user which review type they want. Triggers on "test", "review", "verify", "QA", "check", "audit".
user_invocable: true
---

# Test Automation Skill

Comprehensive testing and review framework supporting both infrastructure health checks and software code reviews.

---

## Step 0: Determine Review Type

**IMPORTANT**: Before proceeding, determine which type of review the user wants.

### Auto-Detection Rules

**Run Infrastructure/Health Review if:**
- User says "test the app", "run tests", "health check", "server status"
- User says "verify deployment", "check production", "QA the site"
- User mentions "server", "database", "API endpoints", "SSL", "services"
- Context involves a recent deployment or production issue

**Run Code Review if:**
- User says "review my code", "check my changes", "audit the code"
- User says "security review", "code quality", "best practices check"
- User mentions "PR", "pull request", "commit", "diff", "changes"
- Context involves recent code modifications

**Ask the User if:**
- The request is ambiguous (e.g., just "review" or "test")
- Both types might be relevant
- You're unsure which is more appropriate

### How to Ask

Use the AskUserQuestion tool:

```
Question: "What type of review would you like me to perform?"
Options:
1. Infrastructure Health Review - Server health, database, APIs, SSL, services, and system checks via SSH
2. Code Review - Security audit, best practices, code quality, and analysis of recent changes
3. Both - Run infrastructure checks first, then review recent code changes
```

---

# PART A: INFRASTRUCTURE / HEALTH REVIEW

## Execution Philosophy

**Ruthlessly automate**: If it CAN be tested via SSH, curl, or CLI - it WILL be automated.
**Clear manual fallback**: For browser-dependent tests, provide exact click-by-click instructions.
**Prioritized tracking**: Maintain a living todo list showing test progress and results.

## Step 1: Initialize Test Session

Create a todo list to track all test categories:

```
Use TodoWrite to create:
1. Server Health Checks (automated)
2. Database Connectivity Tests (automated)
3. API Endpoint Tests (automated)
4. Authentication Flow Tests (automated)
5. Background Job Tests (automated)
6. External Service Tests (automated)
7. UI/Browser Tests (manual - provide checklist)
8. Generate Test Report
```

## Step 2: Detect Project Type & Configuration

Before testing, identify the project stack:

```bash
# Detect project type
ls -la package.json composer.json requirements.txt Gemfile go.mod Cargo.toml 2>/dev/null

# Check for common config files
ls -la .env* config/ 2>/dev/null | head -20

# Check for existing test frameworks
ls -la tests/ spec/ __tests__/ test/ phpunit.xml jest.config.* pytest.ini 2>/dev/null
```

**Supported Stacks** (auto-detected):
- **Laravel/PHP**: composer.json + artisan
- **Node.js**: package.json
- **Python**: requirements.txt or pyproject.toml
- **Rails**: Gemfile + config/routes.rb
- **Go**: go.mod
- **Generic**: HTTP endpoint testing

## Step 3: Server Health Checks (Automated via SSH)

**Prerequisite**: SSH access to production server configured.

### 3.1 System Resources
```bash
ssh $SERVER_USER@$SERVER_IP << 'EOF'
echo "=== SYSTEM HEALTH ==="
echo ""
echo "--- CPU Load ---"
uptime
echo ""
echo "--- Memory Usage ---"
free -h
echo ""
echo "--- Disk Space ---"
df -h | grep -E '^/dev|Filesystem'
echo ""
echo "--- Process Count ---"
ps aux | wc -l
echo ""
echo "--- Top Processes (CPU) ---"
ps aux --sort=-%cpu | head -6
EOF
```

### 3.2 Service Status (adapt to project)
```bash
ssh $SERVER_USER@$SERVER_IP << 'EOF'
echo "=== SERVICE STATUS ==="
echo ""
# Common services - uncomment/modify as needed
systemctl is-active nginx 2>/dev/null && echo "nginx: $(systemctl is-active nginx)" || echo "nginx: not installed"
systemctl is-active apache2 2>/dev/null && echo "apache2: $(systemctl is-active apache2)" || echo "apache2: not installed"
systemctl is-active mysql 2>/dev/null && echo "mysql: $(systemctl is-active mysql)" || echo "mysql: not installed"
systemctl is-active postgresql 2>/dev/null && echo "postgresql: $(systemctl is-active postgresql)" || echo "postgresql: not installed"
systemctl is-active redis 2>/dev/null && echo "redis: $(systemctl is-active redis)" || echo "redis: not installed"
systemctl is-active php*-fpm 2>/dev/null && echo "php-fpm: $(systemctl is-active php*-fpm 2>/dev/null | head -1)" || echo "php-fpm: not installed"

# Docker containers if applicable
if command -v docker &> /dev/null; then
    echo ""
    echo "--- Docker Containers ---"
    docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}" 2>/dev/null || echo "Docker not running"
fi
EOF
```

### 3.3 Application Logs (Last Errors)
```bash
ssh $SERVER_USER@$SERVER_IP << 'EOF'
echo "=== RECENT ERRORS ==="
echo ""
# Laravel
if [ -f /var/www/*/storage/logs/laravel.log ]; then
    echo "--- Laravel Errors (last 50 lines) ---"
    tail -50 /var/www/*/storage/logs/laravel.log 2>/dev/null | grep -i "error\|exception\|fatal" | tail -10
fi
# Nginx
if [ -f /var/log/nginx/error.log ]; then
    echo ""
    echo "--- Nginx Errors (last 10) ---"
    tail -20 /var/log/nginx/error.log 2>/dev/null | grep -i "error" | tail -10
fi
# System logs
echo ""
echo "--- System Errors (last 10) ---"
journalctl -p err -n 10 --no-pager 2>/dev/null || tail -10 /var/log/syslog 2>/dev/null | grep -i error
EOF
```

## Step 4: Database Connectivity Tests (Automated)

### 4.1 Laravel/PHP
```bash
ssh $SERVER_USER@$SERVER_IP << 'EOF'
cd /var/www/$APP_DIR
echo "=== DATABASE TESTS ==="
echo ""
echo "--- Connection Test ---"
php artisan tinker --execute="try { DB::connection()->getPdo(); echo 'SUCCESS: Database connected'; } catch(\Exception \$e) { echo 'FAILED: '.\$e->getMessage(); }"
echo ""
echo "--- Migration Status ---"
php artisan migrate:status | tail -20
echo ""
echo "--- Table Count ---"
php artisan tinker --execute="echo 'Tables: ' . count(DB::select('SHOW TABLES'));"
EOF
```

### 4.2 Generic MySQL/PostgreSQL
```bash
ssh $SERVER_USER@$SERVER_IP << 'EOF'
echo "--- MySQL Connection Test ---"
mysql -u $DB_USER -p$DB_PASS -e "SELECT 1 as connection_test;" 2>/dev/null && echo "MySQL: Connected" || echo "MySQL: Failed"

echo ""
echo "--- PostgreSQL Connection Test ---"
PGPASSWORD=$DB_PASS psql -U $DB_USER -d $DB_NAME -c "SELECT 1 as connection_test;" 2>/dev/null && echo "PostgreSQL: Connected" || echo "PostgreSQL: Failed"
EOF
```

## Step 5: API Endpoint Tests (Automated via curl)

### 5.1 Discover Routes (Laravel)
```bash
ssh $SERVER_USER@$SERVER_IP << 'EOF'
cd /var/www/$APP_DIR
echo "=== ROUTE LIST (API) ==="
php artisan route:list --columns=method,uri,name | grep -E "^.*api|^.*GET|^.*POST" | head -30
EOF
```

### 5.2 HTTP Endpoint Tests
```bash
# Health/status endpoint
echo "=== API ENDPOINT TESTS ==="
echo ""

# Test homepage (200 OK)
echo "--- Homepage ---"
HTTP_CODE=$(curl -s -o /dev/null -w "%{http_code}" https://$DOMAIN/)
if [ "$HTTP_CODE" = "200" ]; then
    echo "PASS: Homepage returns 200"
else
    echo "FAIL: Homepage returns $HTTP_CODE (expected 200)"
fi

# Test common endpoints
echo ""
echo "--- API Health Check ---"
HTTP_CODE=$(curl -s -o /dev/null -w "%{http_code}" https://$DOMAIN/api/health 2>/dev/null)
echo "GET /api/health: $HTTP_CODE"

# Test authentication endpoint
echo ""
echo "--- Auth Endpoints ---"
HTTP_CODE=$(curl -s -o /dev/null -w "%{http_code}" https://$DOMAIN/login)
echo "GET /login: $HTTP_CODE"

HTTP_CODE=$(curl -s -o /dev/null -w "%{http_code}" https://$DOMAIN/register)
echo "GET /register: $HTTP_CODE"

# Test 404 handling
echo ""
echo "--- 404 Handling ---"
HTTP_CODE=$(curl -s -o /dev/null -w "%{http_code}" https://$DOMAIN/this-should-not-exist-12345)
if [ "$HTTP_CODE" = "404" ]; then
    echo "PASS: Returns 404 for missing routes"
else
    echo "WARN: Returns $HTTP_CODE for missing routes (expected 404)"
fi

# Test response times
echo ""
echo "--- Response Times ---"
RESPONSE_TIME=$(curl -s -o /dev/null -w "%{time_total}" https://$DOMAIN/)
echo "Homepage load time: ${RESPONSE_TIME}s"
if (( $(echo "$RESPONSE_TIME < 2.0" | bc -l) )); then
    echo "PASS: Under 2 second threshold"
else
    echo "WARN: Response time exceeds 2 seconds"
fi
```

### 5.3 POST/Form Endpoint Tests (with CSRF)
```bash
# Test form submission endpoints (need to handle CSRF)
echo "=== FORM ENDPOINT TESTS ==="

# Get CSRF token first
CSRF_TOKEN=$(curl -s -c cookies.txt https://$DOMAIN/login | grep -oP 'name="_token" value="\K[^"]+' | head -1)

if [ -n "$CSRF_TOKEN" ]; then
    echo "CSRF Token obtained: ${CSRF_TOKEN:0:20}..."

    # Test login with invalid credentials (should return 422 or redirect)
    HTTP_CODE=$(curl -s -o /dev/null -w "%{http_code}" -b cookies.txt \
        -X POST https://$DOMAIN/login \
        -H "Content-Type: application/x-www-form-urlencoded" \
        -d "_token=$CSRF_TOKEN&email=test@invalid.com&password=wrongpassword")
    echo "POST /login (invalid): $HTTP_CODE (expected 422 or 302)"
else
    echo "WARN: Could not obtain CSRF token"
fi

rm -f cookies.txt
```

## Step 6: Authentication Flow Tests (Automated)

### 6.1 Session Handling
```bash
echo "=== AUTHENTICATION TESTS ==="
echo ""

# Test unauthenticated access to protected route
echo "--- Protected Route Access ---"
HTTP_CODE=$(curl -s -o /dev/null -w "%{http_code}" https://$DOMAIN/dashboard)
if [ "$HTTP_CODE" = "302" ] || [ "$HTTP_CODE" = "401" ]; then
    echo "PASS: Protected route redirects unauthenticated users ($HTTP_CODE)"
else
    echo "FAIL: Protected route returns $HTTP_CODE (expected 302 or 401)"
fi

# Test cookie security headers
echo ""
echo "--- Security Headers ---"
HEADERS=$(curl -sI https://$DOMAIN/)
echo "$HEADERS" | grep -i "strict-transport-security" && echo "PASS: HSTS enabled" || echo "WARN: HSTS not found"
echo "$HEADERS" | grep -i "x-frame-options" && echo "PASS: X-Frame-Options set" || echo "WARN: X-Frame-Options not set"
echo "$HEADERS" | grep -i "x-content-type-options" && echo "PASS: X-Content-Type-Options set" || echo "INFO: X-Content-Type-Options not set"
```

### 6.2 Laravel Auth Tests (via artisan)
```bash
ssh $SERVER_USER@$SERVER_IP << 'EOF'
cd /var/www/$APP_DIR
echo "=== LARAVEL AUTH TESTS ==="
echo ""

# Check user count
echo "--- User Statistics ---"
php artisan tinker --execute="echo 'Total Users: ' . \App\Models\User::count();"
php artisan tinker --execute="echo 'Recent Users (24h): ' . \App\Models\User::where('created_at', '>', now()->subDay())->count();"

# Check for locked accounts or issues
echo ""
echo "--- Auth Health ---"
php artisan tinker --execute="echo 'Failed Jobs: ' . DB::table('failed_jobs')->count();" 2>/dev/null || echo "No failed_jobs table"
EOF
```

## Step 7: Background Job Tests (Automated)

### 7.1 Queue Status
```bash
ssh $SERVER_USER@$SERVER_IP << 'EOF'
cd /var/www/$APP_DIR
echo "=== QUEUE/JOB TESTS ==="
echo ""

# Laravel queues
echo "--- Queue Status ---"
php artisan queue:monitor 2>/dev/null || echo "Queue monitor not available"

echo ""
echo "--- Pending Jobs ---"
php artisan tinker --execute="echo 'Pending: ' . DB::table('jobs')->count();" 2>/dev/null || echo "No jobs table"

echo ""
echo "--- Failed Jobs ---"
php artisan queue:failed 2>/dev/null | head -10 || echo "No failed jobs or table"

# Scheduler
echo ""
echo "--- Scheduled Tasks ---"
php artisan schedule:list 2>/dev/null | head -20 || echo "No scheduled tasks"
EOF
```

### 7.2 Worker Process Check
```bash
ssh $SERVER_USER@$SERVER_IP << 'EOF'
echo "--- Queue Workers ---"
ps aux | grep -E "queue:work|queue:listen|artisan" | grep -v grep || echo "No queue workers running"

# Supervisor status (if used)
supervisorctl status 2>/dev/null || echo "Supervisor not installed/running"

# Systemd queue services
systemctl list-units --type=service | grep -i queue 2>/dev/null || echo "No queue services found"
EOF
```

## Step 8: External Service Tests (Automated)

### 8.1 Third-Party API Connectivity
```bash
echo "=== EXTERNAL SERVICE TESTS ==="
echo ""

# Test Stripe API (if configured)
echo "--- Stripe API ---"
curl -s -o /dev/null -w "%{http_code}" https://api.stripe.com/v1/balance \
    -u "$STRIPE_SECRET_KEY:" 2>/dev/null | grep -q "200\|401" && echo "PASS: Stripe API reachable" || echo "WARN: Stripe API issue"

# Test email service (Mailgun/SendGrid/etc)
echo ""
echo "--- Mail Service ---"
# Adapt based on provider

# Test OpenRouter/OpenAI (if used)
echo ""
echo "--- AI API ---"
curl -s -o /dev/null -w "%{http_code}" https://openrouter.ai/api/v1/models \
    -H "Authorization: Bearer $OPENROUTER_KEY" 2>/dev/null | grep -q "200" && echo "PASS: OpenRouter API reachable" || echo "INFO: OpenRouter not configured or unreachable"
```

### 8.2 SSL Certificate Check
```bash
echo "=== SSL CERTIFICATE ==="
echo ""
CERT_EXPIRY=$(echo | openssl s_client -servername $DOMAIN -connect $DOMAIN:443 2>/dev/null | openssl x509 -noout -enddate 2>/dev/null | cut -d= -f2)
if [ -n "$CERT_EXPIRY" ]; then
    echo "Certificate expires: $CERT_EXPIRY"
    DAYS_LEFT=$(( ($(date -d "$CERT_EXPIRY" +%s) - $(date +%s)) / 86400 ))
    if [ $DAYS_LEFT -gt 30 ]; then
        echo "PASS: $DAYS_LEFT days until expiry"
    elif [ $DAYS_LEFT -gt 7 ]; then
        echo "WARN: Only $DAYS_LEFT days until expiry"
    else
        echo "CRITICAL: Only $DAYS_LEFT days until expiry!"
    fi
else
    echo "WARN: Could not check SSL certificate"
fi
```

## Step 9: UI/Browser Tests (Manual Checklist)

**These tests require a real browser and cannot be automated via SSH.**

Output this checklist for the user to complete manually:

---

### Manual Browser Test Checklist

**Environment**: Production / Staging
**Browser**: Chrome / Firefox / Safari
**Date**: [Current Date]
**Tester**: [Name]

#### Homepage & Navigation
- [ ] Homepage loads completely without console errors
- [ ] All navigation links work
- [ ] Logo links to homepage
- [ ] Mobile menu opens/closes correctly
- [ ] Footer links work

#### Authentication Flow
- [ ] Login page renders correctly
- [ ] Login with valid credentials works
- [ ] Login with invalid credentials shows error message
- [ ] "Forgot password" link works
- [ ] Registration form validates required fields
- [ ] Registration creates account successfully
- [ ] Logout works and redirects appropriately
- [ ] Session persists across page refreshes

#### Core Feature Flow
*(Customize per project)*
- [ ] Main feature/product list loads
- [ ] Individual item/detail page works
- [ ] Create new item (if applicable)
- [ ] Edit existing item (if applicable)
- [ ] Delete item with confirmation (if applicable)
- [ ] Search functionality works
- [ ] Filtering/sorting works

#### Forms & Validation
- [ ] Required field validation shows errors
- [ ] Email format validation works
- [ ] Form submits successfully with valid data
- [ ] Success messages display correctly
- [ ] Error messages are clear and helpful

#### Responsive Design
- [ ] Desktop layout (1920px) looks correct
- [ ] Tablet layout (768px) looks correct
- [ ] Mobile layout (375px) looks correct
- [ ] Touch targets are appropriately sized
- [ ] No horizontal scrolling on mobile

#### Payment/Checkout (if applicable)
- [ ] Pricing page displays correctly
- [ ] Checkout button initiates payment flow
- [ ] Test card (4242 4242 4242 4242) processes
- [ ] Success/confirmation page shows
- [ ] Subscription status updates in account

#### Performance (Manual Observation)
- [ ] Pages load within 3 seconds
- [ ] No visible layout shift during load
- [ ] Images load progressively
- [ ] No frozen/unresponsive UI

#### Browser Console
- [ ] No JavaScript errors in console
- [ ] No 404 errors for assets
- [ ] No CORS errors
- [ ] No mixed content warnings (HTTP on HTTPS)

---

## Step 10: Generate Infrastructure Test Report

After all tests complete, generate summary:

```markdown
# Infrastructure Health Report

**Project**: [Project Name]
**Date**: [Date/Time]
**Environment**: Production
**Triggered By**: [User/Deployment]

## Summary

| Category | Status | Automated | Manual |
|----------|--------|-----------|--------|
| Server Health | PASS/FAIL | Yes | - |
| Database | PASS/FAIL | Yes | - |
| API Endpoints | PASS/FAIL | Yes | - |
| Authentication | PASS/FAIL | Yes | - |
| Background Jobs | PASS/FAIL | Yes | - |
| External Services | PASS/FAIL | Yes | - |
| SSL Certificate | PASS/FAIL | Yes | - |
| UI/Browser | PENDING | - | Yes |

## Critical Issues
1. [List any failures]

## Warnings
1. [List any warnings]

## Recommendations
1. [Action items based on results]
```

---

# PART B: CODE REVIEW

Systematically review recent implementation work for quality, security, and correctness.

## Step 1: Identify Recent Changes

First, check git for modifications:

```bash
git diff HEAD~3 --name-only
git log --oneline -5
git status
```

Focus on recently modified files, especially:
- Controllers (authorization, validation)
- Views/templates (XSS, CSRF)
- JavaScript (DOM manipulation, API calls)
- Routes (middleware)
- Services (business logic)
- Migrations (database changes)

## Step 2: Security Checklist

### PHP/Laravel Security
- [ ] Input validation with proper rules and max lengths
- [ ] Authorization checks (ownership verification, `canBeViewedBy()`, policies)
- [ ] CSRF protection on forms (`@csrf`)
- [ ] No raw SQL queries (use Eloquent/Query Builder with bindings)
- [ ] Sensitive data not logged (passwords, tokens, API keys)
- [ ] Mass assignment protection (fillable/guarded)
- [ ] No hardcoded credentials or secrets
- [ ] Rate limiting on sensitive endpoints
- [ ] Proper authentication middleware on protected routes

### JavaScript Security
- [ ] No `innerHTML` with unsanitized user input (use `textContent`)
- [ ] API calls include CSRF token in headers
- [ ] No exposed secrets or API keys in client-side code
- [ ] Error handling doesn't leak sensitive info to console
- [ ] No `eval()` or `new Function()` with user input
- [ ] Proper escaping when building URLs with user data

### Blade Templates Security
- [ ] Use `{{ }}` for output (auto-escapes HTML)
- [ ] Use `{!! !!}` only for trusted HTML (sanitized server-side)
- [ ] Forms have `@csrf` directive
- [ ] No user input directly in `onclick`, `href="javascript:"`, etc.

### Database Security
- [ ] No SQL injection vulnerabilities
- [ ] Migrations don't expose sensitive defaults
- [ ] Foreign key constraints where appropriate
- [ ] Indexes on frequently queried columns

## Step 3: Best Practices Checklist

### Code Quality
- [ ] No dead code or unused imports/variables
- [ ] No broken function/method references
- [ ] Consistent naming conventions (camelCase, snake_case per language)
- [ ] Proper error handling with user-friendly messages
- [ ] No `console.log` or `dd()` in production code
- [ ] No commented-out code blocks (remove or document why kept)
- [ ] Methods/functions have single responsibility
- [ ] No magic numbers (use constants/config)

### Efficiency
- [ ] No N+1 query problems (use eager loading: `with()`)
- [ ] Appropriate caching where beneficial
- [ ] No unnecessary database calls in loops
- [ ] Reasonable AJAX payload sizes
- [ ] Database queries use proper indexes
- [ ] No redundant data fetching

### Laravel Specific
- [ ] Form requests for complex validation
- [ ] Route model binding where appropriate
- [ ] Service layer for complex business logic
- [ ] Jobs for background processing (not in request cycle)
- [ ] Events for decoupled side effects
- [ ] Proper use of config/env (no `env()` outside config files)

### Frontend Specific
- [ ] CSS follows project conventions
- [ ] No inline styles that should be classes
- [ ] Responsive design considerations
- [ ] Accessibility basics (alt text, labels, focus states)
- [ ] Loading states for async operations

## Step 4: Functional Verification

- [ ] All referenced functions/methods exist and are spelled correctly
- [ ] All routes properly defined in routes files
- [ ] All required views/templates exist
- [ ] JavaScript event handlers properly bound
- [ ] No typos in method/function names
- [ ] Imports/requires point to existing files
- [ ] Database columns referenced actually exist

## Step 5: Generate Code Review Report

Provide summary table:

```markdown
# Code Review Report

**Files Reviewed**: [List files]
**Reviewer**: Claude
**Date**: [Date]

## Summary

| Category | Status | Issues Found |
|----------|--------|--------------|
| Security | PASS/FAIL | [Count] |
| Best Practices | PASS/FAIL | [Count] |
| Efficiency | PASS/FAIL | [Count] |
| Broken References | PASS/FAIL | [Count] |

## Issues by Severity

### Critical (Must fix before deploy)
1. [Security vulnerability, broken functionality, data loss risk]

### Warnings (Should fix soon)
1. [Performance issues, code smell, minor security concerns]

### Suggestions (Nice to have)
1. [Style improvements, refactoring opportunities]

## Files Reviewed

| File | Lines Changed | Issues |
|------|---------------|--------|
| path/to/file.php | +50/-10 | 2 warnings |

## Detailed Findings

### [file_path:line_number] - Issue Title
**Severity**: Critical/Warning/Suggestion
**Category**: Security/Performance/Best Practice
**Description**: What the issue is
**Recommendation**: How to fix it
```

---

# PART C: COMBINED REVIEW

When running both reviews:

1. Run Infrastructure Health Review first (server, database, APIs)
2. Then run Code Review on recent changes
3. Combine findings into unified report
4. Prioritize issues across both domains

## Combined Report Format

```markdown
# Combined Test & Review Report

**Project**: [Name]
**Date**: [Date]

## Executive Summary
[1-2 sentence overview of overall health]

## Infrastructure Health
| Check | Status |
|-------|--------|
| Server | PASS/WARN/FAIL |
| Database | PASS/WARN/FAIL |
| APIs | PASS/WARN/FAIL |
| SSL | PASS/WARN/FAIL |

## Code Quality
| Check | Status |
|-------|--------|
| Security | PASS/WARN/FAIL |
| Best Practices | PASS/WARN/FAIL |
| Performance | PASS/WARN/FAIL |

## Critical Issues (Fix Immediately)
1. [Combined list from both reviews]

## Warnings (Fix Soon)
1. [Combined list]

## Manual Tests Required
[Browser test checklist if applicable]
```

---

## Customization Variables

Set these before running tests:

```bash
# Server access (for infrastructure review)
export SERVER_USER="root"           # or deploy user
export SERVER_IP="your.server.ip"
export APP_DIR="your-app"           # directory name under /var/www/

# Application
export DOMAIN="yourdomain.com"

# Database (if testing directly)
export DB_USER="dbuser"
export DB_PASS="dbpass"
export DB_NAME="dbname"

# API Keys (for external service tests)
export STRIPE_SECRET_KEY="sk_test_..."
export OPENROUTER_KEY="sk-..."
```

## Quick Run Commands

```bash
# Full infrastructure test suite
/test-automation infrastructure

# Code review only
/test-automation code

# Both reviews
/test-automation full

# If ambiguous, will ask which type
/test-automation
```

## When to Run Each Type

### Infrastructure Review
- After deployments
- When investigating production issues
- Daily/weekly health checks
- Before major releases

### Code Review
- After completing implementation work
- Before creating pull requests
- After refactoring code
- When debugging unexpected behavior

### Both
- Major releases
- Post-incident analysis
- Periodic comprehensive audits
