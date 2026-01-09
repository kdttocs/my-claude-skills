---
name: ui-test
description: Automated UI testing using Playwright. Opens browsers, navigates URLs, clicks through UI flows, fills forms, takes screenshots, and tests complete user journeys. Use after frontend changes to verify the UI works correctly. Triggers on "ui test", "test the ui", "browser test", "screenshot test", "test the frontend", "visual test".
user_invocable: true
---

# UI Test Skill

Automated browser-based UI testing using Playwright. Tests user flows, captures screenshots, fills forms, and verifies the complete user experience.

## Prerequisites

### Install Playwright (one-time setup)

```bash
# Install Playwright and browsers
npm init -y 2>/dev/null || true
npm install -D playwright @playwright/test
npx playwright install chromium

# Verify installation
npx playwright --version
```

## Step 1: Determine Test Scope

Before running tests, clarify with the user:

1. **What URLs to test?** (homepage, dashboard, specific pages)
2. **What flows to test?** (signup, login, form submission, checkout)
3. **What to capture?** (screenshots, errors, performance)
4. **What viewport?** (desktop, tablet, mobile, all)

Use AskUserQuestion if needed:

```
Question: "What would you like me to test?"
Options:
1. Full user journey (signup → login → main features → logout)
2. Specific page screenshots (capture current state of key pages)
3. Form validation testing (test form inputs and error states)
4. Responsive design (test across desktop, tablet, mobile viewports)
```

## Step 2: Create Test Script

Generate a Playwright test script based on the project and test requirements.

### Base Test Template

```javascript
// ui-test.js - Generated UI Test Script
const { chromium } = require('playwright');
const fs = require('fs');
const path = require('path');

// Configuration
const CONFIG = {
    baseUrl: process.env.TEST_URL || 'https://your-domain.com',
    screenshotDir: './ui-test-screenshots',
    viewports: {
        desktop: { width: 1920, height: 1080 },
        tablet: { width: 768, height: 1024 },
        mobile: { width: 375, height: 812 }
    },
    timeout: 30000,
    slowMo: 100 // Slow down for visibility
};

// Ensure screenshot directory exists
if (!fs.existsSync(CONFIG.screenshotDir)) {
    fs.mkdirSync(CONFIG.screenshotDir, { recursive: true });
}

// Test results tracking
const results = {
    passed: [],
    failed: [],
    screenshots: [],
    startTime: new Date(),
    endTime: null
};

// Helper: Take screenshot with timestamp
async function screenshot(page, name) {
    const timestamp = new Date().toISOString().replace(/[:.]/g, '-');
    const filename = `${name}-${timestamp}.png`;
    const filepath = path.join(CONFIG.screenshotDir, filename);
    await page.screenshot({ path: filepath, fullPage: true });
    results.screenshots.push({ name, filepath, timestamp });
    console.log(`📸 Screenshot: ${filename}`);
    return filepath;
}

// Helper: Log test result
function logResult(testName, passed, error = null) {
    if (passed) {
        results.passed.push(testName);
        console.log(`✅ PASS: ${testName}`);
    } else {
        results.failed.push({ test: testName, error: error?.message || 'Unknown error' });
        console.log(`❌ FAIL: ${testName} - ${error?.message || 'Unknown error'}`);
    }
}

// Helper: Wait for network idle
async function waitForLoad(page) {
    await page.waitForLoadState('networkidle', { timeout: CONFIG.timeout });
}

// Helper: Safe click with retry
async function safeClick(page, selector, options = {}) {
    try {
        await page.waitForSelector(selector, { timeout: options.timeout || 5000 });
        await page.click(selector);
        return true;
    } catch (e) {
        console.log(`⚠️ Could not click: ${selector}`);
        return false;
    }
}

// Helper: Safe fill with validation
async function safeFill(page, selector, value, options = {}) {
    try {
        await page.waitForSelector(selector, { timeout: options.timeout || 5000 });
        await page.fill(selector, value);
        return true;
    } catch (e) {
        console.log(`⚠️ Could not fill: ${selector}`);
        return false;
    }
}

// Helper: Check for console errors
function setupConsoleMonitor(page) {
    const errors = [];
    page.on('console', msg => {
        if (msg.type() === 'error') {
            errors.push(msg.text());
        }
    });
    page.on('pageerror', err => {
        errors.push(err.message);
    });
    return errors;
}

// Main test runner
async function runTests() {
    console.log('🚀 Starting UI Tests...\n');
    console.log(`Base URL: ${CONFIG.baseUrl}`);
    console.log(`Screenshot Dir: ${CONFIG.screenshotDir}\n`);

    const browser = await chromium.launch({
        headless: process.env.HEADLESS !== 'false',
        slowMo: CONFIG.slowMo
    });

    try {
        // Run tests for each viewport
        for (const [viewportName, viewport] of Object.entries(CONFIG.viewports)) {
            console.log(`\n📱 Testing viewport: ${viewportName} (${viewport.width}x${viewport.height})\n`);

            const context = await browser.newContext({
                viewport,
                userAgent: viewportName === 'mobile'
                    ? 'Mozilla/5.0 (iPhone; CPU iPhone OS 14_0 like Mac OS X) AppleWebKit/605.1.15'
                    : undefined
            });
            const page = await context.newPage();
            const consoleErrors = setupConsoleMonitor(page);

            // ========================================
            // TEST SUITE - Customize per project
            // ========================================

            // Test 1: Homepage loads
            try {
                await page.goto(CONFIG.baseUrl, { waitUntil: 'networkidle' });
                await screenshot(page, `01-homepage-${viewportName}`);
                logResult(`Homepage loads (${viewportName})`, true);
            } catch (e) {
                await screenshot(page, `01-homepage-error-${viewportName}`);
                logResult(`Homepage loads (${viewportName})`, false, e);
            }

            // Test 2: Navigation works
            try {
                // Click main navigation links
                const navLinks = await page.$$('nav a, header a');
                console.log(`Found ${navLinks.length} navigation links`);
                logResult(`Navigation elements found (${viewportName})`, navLinks.length > 0);
            } catch (e) {
                logResult(`Navigation elements (${viewportName})`, false, e);
            }

            // Test 3: Login page
            try {
                await page.goto(`${CONFIG.baseUrl}/login`, { waitUntil: 'networkidle' });
                await screenshot(page, `02-login-${viewportName}`);

                // Check for login form elements
                const emailField = await page.$('input[type="email"], input[name="email"]');
                const passwordField = await page.$('input[type="password"]');
                const submitBtn = await page.$('button[type="submit"], input[type="submit"]');

                logResult(`Login form elements (${viewportName})`,
                    emailField && passwordField && submitBtn);
            } catch (e) {
                logResult(`Login page (${viewportName})`, false, e);
            }

            // Test 4: Form validation
            try {
                // Try submitting empty form
                await safeClick(page, 'button[type="submit"]');
                await page.waitForTimeout(500);
                await screenshot(page, `03-form-validation-${viewportName}`);

                // Check for validation messages
                const validationErrors = await page.$$('.error, .invalid, [class*="error"], [class*="invalid"]');
                console.log(`Found ${validationErrors.length} validation indicators`);
                logResult(`Form validation shows errors (${viewportName})`, true);
            } catch (e) {
                logResult(`Form validation (${viewportName})`, false, e);
            }

            // Test 5: Check for console errors
            if (consoleErrors.length > 0) {
                console.log(`\n⚠️ Console errors on ${viewportName}:`);
                consoleErrors.forEach(err => console.log(`   - ${err}`));
                logResult(`No console errors (${viewportName})`, false,
                    new Error(`${consoleErrors.length} console errors`));
            } else {
                logResult(`No console errors (${viewportName})`, true);
            }

            await context.close();
        }

    } finally {
        await browser.close();
    }

    // Generate report
    results.endTime = new Date();
    generateReport();
}

// Generate test report
function generateReport() {
    const duration = (results.endTime - results.startTime) / 1000;

    console.log('\n' + '='.repeat(60));
    console.log('📊 UI TEST REPORT');
    console.log('='.repeat(60));
    console.log(`Duration: ${duration.toFixed(2)}s`);
    console.log(`Passed: ${results.passed.length}`);
    console.log(`Failed: ${results.failed.length}`);
    console.log(`Screenshots: ${results.screenshots.length}`);
    console.log('');

    if (results.failed.length > 0) {
        console.log('❌ FAILED TESTS:');
        results.failed.forEach(f => {
            console.log(`   - ${f.test}: ${f.error}`);
        });
        console.log('');
    }

    console.log('📸 SCREENSHOTS:');
    results.screenshots.forEach(s => {
        console.log(`   - ${s.filepath}`);
    });
    console.log('');

    // Save JSON report
    const reportPath = path.join(CONFIG.screenshotDir, 'report.json');
    fs.writeFileSync(reportPath, JSON.stringify(results, null, 2));
    console.log(`📄 Report saved: ${reportPath}`);

    // Exit with error code if tests failed
    if (results.failed.length > 0) {
        console.log('\n❌ UI Tests FAILED');
        process.exit(1);
    } else {
        console.log('\n✅ All UI Tests PASSED');
    }
}

// Run
runTests().catch(console.error);
```

## Step 3: Project-Specific Test Flows

Customize the test script based on the project. Common flows:

### Authentication Flow

```javascript
// Test: Full authentication flow
async function testAuthFlow(page, viewport) {
    console.log('🔐 Testing authentication flow...');

    // Go to login
    await page.goto(`${CONFIG.baseUrl}/login`);
    await screenshot(page, `auth-01-login-page-${viewport}`);

    // Fill login form with test credentials
    await safeFill(page, 'input[name="email"]', 'test@example.com');
    await safeFill(page, 'input[name="password"]', 'testpassword');
    await screenshot(page, `auth-02-filled-form-${viewport}`);

    // Submit
    await safeClick(page, 'button[type="submit"]');
    await waitForLoad(page);
    await screenshot(page, `auth-03-after-submit-${viewport}`);

    // Check if redirected to dashboard or shows error
    const currentUrl = page.url();
    const isLoggedIn = currentUrl.includes('dashboard') ||
                       await page.$('[data-user], .user-menu, .logout');

    logResult(`Authentication flow (${viewport})`, isLoggedIn);
}
```

### Form Submission Flow

```javascript
// Test: Form submission with validation
async function testFormSubmission(page, viewport, formUrl, formData) {
    console.log('📝 Testing form submission...');

    await page.goto(formUrl);
    await screenshot(page, `form-01-empty-${viewport}`);

    // Fill form fields
    for (const [selector, value] of Object.entries(formData)) {
        await safeFill(page, selector, value);
    }
    await screenshot(page, `form-02-filled-${viewport}`);

    // Submit
    await safeClick(page, 'button[type="submit"]');
    await waitForLoad(page);
    await screenshot(page, `form-03-submitted-${viewport}`);

    // Check for success message or redirect
    const successIndicator = await page.$('.success, .alert-success, [class*="success"]');
    logResult(`Form submission (${viewport})`, !!successIndicator);
}
```

### E-commerce Checkout Flow

```javascript
// Test: Add to cart and checkout
async function testCheckoutFlow(page, viewport) {
    console.log('🛒 Testing checkout flow...');

    // Browse products
    await page.goto(`${CONFIG.baseUrl}/products`);
    await screenshot(page, `checkout-01-products-${viewport}`);

    // Click first product
    await safeClick(page, '.product-card, [data-product]');
    await waitForLoad(page);
    await screenshot(page, `checkout-02-product-detail-${viewport}`);

    // Add to cart
    await safeClick(page, '[data-add-to-cart], .add-to-cart, button:has-text("Add to Cart")');
    await page.waitForTimeout(1000);
    await screenshot(page, `checkout-03-added-to-cart-${viewport}`);

    // Go to cart
    await safeClick(page, '[data-cart], .cart-icon, a[href*="cart"]');
    await waitForLoad(page);
    await screenshot(page, `checkout-04-cart-${viewport}`);

    // Proceed to checkout
    await safeClick(page, '[data-checkout], .checkout-btn, button:has-text("Checkout")');
    await waitForLoad(page);
    await screenshot(page, `checkout-05-checkout-${viewport}`);

    logResult(`Checkout flow navigable (${viewport})`, true);
}
```

### Responsive Design Test

```javascript
// Test: Responsive breakpoints
async function testResponsive(browser, url) {
    console.log('📱 Testing responsive design...');

    const breakpoints = [
        { name: 'mobile-sm', width: 320, height: 568 },
        { name: 'mobile', width: 375, height: 812 },
        { name: 'tablet', width: 768, height: 1024 },
        { name: 'laptop', width: 1366, height: 768 },
        { name: 'desktop', width: 1920, height: 1080 },
        { name: 'ultrawide', width: 2560, height: 1440 }
    ];

    for (const bp of breakpoints) {
        const context = await browser.newContext({ viewport: bp });
        const page = await context.newPage();

        await page.goto(url);
        await screenshot(page, `responsive-${bp.name}`);

        // Check for horizontal scroll (bad)
        const hasHorizontalScroll = await page.evaluate(() => {
            return document.documentElement.scrollWidth > document.documentElement.clientWidth;
        });

        if (hasHorizontalScroll) {
            console.log(`⚠️ Horizontal scroll detected at ${bp.name}`);
        }

        await context.close();
    }

    logResult('Responsive screenshots captured', true);
}
```

## Step 4: Run Tests

Execute the test script:

```bash
# Run with visible browser (for debugging)
HEADLESS=false TEST_URL=https://your-domain.com node ui-test.js

# Run headless (for CI)
TEST_URL=https://your-domain.com node ui-test.js

# Run with specific URL
TEST_URL=http://localhost:3000 node ui-test.js
```

## Step 5: Review Results

After tests complete:

1. **Check console output** for pass/fail status
2. **Review screenshots** in `./ui-test-screenshots/`
3. **Check report.json** for detailed results
4. **View failing tests** and their error messages

## Step 6: Server-Side Debugging (When Issues Found)

**IMPORTANT**: If UI tests fail or show unexpected behavior, check the project's `CLAUDE.md` file for SSH access instructions. If SSH access is available, connect to the server to diagnose the root cause.

### Check CLAUDE.md for SSH Access

Look for patterns like:
- `ssh root@IP_ADDRESS` or `ssh user@hostname`
- Server IP addresses or hostnames
- Application paths (e.g., `/var/www/app-name`)
- Log file locations
- Database access commands

### Server Diagnostic Commands

When UI issues are detected, SSH into the server and run these diagnostics:

#### 1. Check Application Logs (Recent Errors)

```bash
# Laravel
ssh $SERVER_USER@$SERVER_IP << 'EOF'
echo "=== RECENT APPLICATION ERRORS ==="
tail -100 /var/www/$APP_DIR/storage/logs/laravel.log | grep -i "error\|exception\|fatal" | tail -20
echo ""
echo "=== LAST 10 LOG ENTRIES ==="
tail -20 /var/www/$APP_DIR/storage/logs/laravel.log
EOF

# Node.js/PM2
ssh $SERVER_USER@$SERVER_IP "pm2 logs --lines 50"

# Generic
ssh $SERVER_USER@$SERVER_IP "tail -50 /var/log/app/error.log"
```

#### 2. Check Web Server Logs

```bash
ssh $SERVER_USER@$SERVER_IP << 'EOF'
echo "=== NGINX ERRORS ==="
tail -30 /var/log/nginx/error.log | grep -v "favicon"

echo ""
echo "=== RECENT 500 ERRORS ==="
grep " 500 " /var/log/nginx/access.log | tail -10

echo ""
echo "=== RECENT 404s ==="
grep " 404 " /var/log/nginx/access.log | tail -10
EOF
```

#### 3. Check Database for Related Data

```bash
# Laravel/Artisan
ssh $SERVER_USER@$SERVER_IP << 'EOF'
cd /var/www/$APP_DIR

echo "=== RECENT FAILED JOBS ==="
php artisan queue:failed 2>/dev/null | head -10

echo ""
echo "=== DATABASE CONNECTIVITY ==="
php artisan tinker --execute="try { DB::connection()->getPdo(); echo 'Connected'; } catch(\Exception \$e) { echo 'FAILED: '.\$e->getMessage(); }"

echo ""
echo "=== RECENT USER ACTIVITY ==="
php artisan tinker --execute="\$users = App\Models\User::latest()->take(5)->get(['id','email','created_at']); foreach(\$users as \$u) echo \$u->id . ' | ' . \$u->email . ' | ' . \$u->created_at . PHP_EOL;"
EOF

# Direct MySQL query
ssh $SERVER_USER@$SERVER_IP "mysql -u \$DB_USER -p\$DB_PASS \$DB_NAME -e 'SELECT * FROM failed_jobs ORDER BY id DESC LIMIT 5;'"
```

#### 4. Check Service Status

```bash
ssh $SERVER_USER@$SERVER_IP << 'EOF'
echo "=== SERVICE STATUS ==="
systemctl is-active nginx php*-fpm mysql redis-server 2>/dev/null || echo "Some services not found"

echo ""
echo "=== DISK SPACE ==="
df -h | grep -E "/$|/var"

echo ""
echo "=== MEMORY ==="
free -h

echo ""
echo "=== LOAD AVERAGE ==="
uptime
EOF
```

#### 5. Check for Recent Deployments

```bash
ssh $SERVER_USER@$SERVER_IP << 'EOF'
cd /var/www/$APP_DIR
echo "=== RECENT COMMITS ==="
git log --oneline -5

echo ""
echo "=== LAST DEPLOYMENT TIME ==="
stat -c '%y' .git/FETCH_HEAD 2>/dev/null || echo "Unknown"
EOF
```

### Correlating UI Failures with Server Issues

| UI Symptom | Server Check | Common Cause |
|------------|--------------|--------------|
| Page timeout | Check nginx/php-fpm status, memory | Service crashed, OOM |
| 500 error | Application logs, database | Exception, DB connection |
| Missing data | Database queries, recent migrations | Migration not run, cache stale |
| Slow load | Load average, slow query log | High traffic, unoptimized query |
| Form submission fails | Failed jobs, queue worker | Queue not processing |
| Assets not loading | Nginx config, disk space | Disk full, misconfigured |
| Session issues | Redis/session driver, cookies | Session driver down |

### Automated Server Diagnostic Script

When UI tests fail, automatically run this diagnostic:

```bash
#!/bin/bash
# server-diagnostic.sh - Run after UI test failures

SERVER_USER="${SERVER_USER:-root}"
SERVER_IP="${SERVER_IP:-your.server.ip}"
APP_DIR="${APP_DIR:-your-app}"

echo "🔍 Running server diagnostics for UI failure..."
echo ""

ssh $SERVER_USER@$SERVER_IP << EOF
echo "=== 1. SERVICE HEALTH ==="
systemctl is-active nginx php*-fpm mysql 2>/dev/null | paste - - -

echo ""
echo "=== 2. RECENT ERRORS (last 5 min) ==="
find /var/www/$APP_DIR/storage/logs -name "*.log" -mmin -5 -exec tail -10 {} \;

echo ""
echo "=== 3. MEMORY STATUS ==="
free -h | head -2

echo ""
echo "=== 4. FAILED JOBS ==="
cd /var/www/$APP_DIR && php artisan tinker --execute="echo DB::table('failed_jobs')->count() . ' failed jobs';" 2>/dev/null

echo ""
echo "=== 5. QUEUE STATUS ==="
cd /var/www/$APP_DIR && php artisan tinker --execute="echo DB::table('jobs')->count() . ' pending jobs';" 2>/dev/null

echo ""
echo "=== 6. LAST DEPLOY ==="
cd /var/www/$APP_DIR && git log --oneline -1
EOF

echo ""
echo "✅ Server diagnostics complete"
```

### When to Run Server Diagnostics

Run server-side checks when:

1. **Page returns 500/502/503** - Check app logs and service status
2. **Page times out** - Check memory, load, and service health
3. **Form submission fails silently** - Check queue workers and failed jobs
4. **Data not appearing** - Check database and recent migrations
5. **Intermittent failures** - Check error rates in logs
6. **After deployment** - Verify services restarted correctly

### Example: Full Debug Flow

```
1. UI Test detects: Login form submission returns 500 error
   ↓
2. Take screenshot of error state
   ↓
3. SSH to server (using CLAUDE.md credentials)
   ↓
4. Check Laravel log:
   tail -50 /var/www/app/storage/logs/laravel.log | grep -i error
   ↓
5. Find: "SQLSTATE[HY000] [2002] Connection refused"
   ↓
6. Check MySQL:
   systemctl status mysql  → Active: inactive (dead)
   ↓
7. Root cause: MySQL crashed
   ↓
8. Fix: systemctl start mysql
   ↓
9. Re-run UI test to verify fix
```

## Quick Commands

```bash
# Install (one-time)
npm install -D playwright && npx playwright install chromium

# Set your base URL (required)
export TEST_URL="https://your-domain.com"

# Test production
TEST_URL=$TEST_URL node ui-test.js

# Test staging
TEST_URL="https://staging.${TEST_URL#https://}" node ui-test.js

# Test local
TEST_URL=http://localhost:8000 node ui-test.js

# Debug mode (visible browser)
HEADLESS=false TEST_URL=$TEST_URL node ui-test.js
```

**Note**: Set `TEST_URL` to your production domain. The test script uses this environment variable to determine which site to test.

## Common Test Patterns

### Check element exists
```javascript
const element = await page.$('selector');
logResult('Element exists', !!element);
```

### Check text content
```javascript
const text = await page.textContent('selector');
logResult('Correct text', text.includes('expected'));
```

### Check element visible
```javascript
const visible = await page.isVisible('selector');
logResult('Element visible', visible);
```

### Check URL after navigation
```javascript
await page.click('a');
await waitForLoad(page);
logResult('Navigated correctly', page.url().includes('/expected-path'));
```

### Check form validation message
```javascript
await page.fill('input[name="email"]', 'invalid');
await page.click('button[type="submit"]');
const errorMsg = await page.$('.error-message');
logResult('Validation message shown', !!errorMsg);
```

### Take screenshot on error
```javascript
try {
    // test code
} catch (e) {
    await screenshot(page, 'error-state');
    throw e;
}
```

## Customization Variables

Set these environment variables:

```bash
export TEST_URL="https://your-domain.com"
export HEADLESS="true"  # or "false" for visible browser
export SLOW_MO="100"    # milliseconds between actions
```

## Integration with CI/CD

Add to GitHub Actions:

```yaml
- name: Run UI Tests
  run: |
    npm install -D playwright
    npx playwright install chromium
    TEST_URL=${{ secrets.TEST_URL }} node ui-test.js

- name: Upload Screenshots
  uses: actions/upload-artifact@v3
  if: always()
  with:
    name: ui-screenshots
    path: ./ui-test-screenshots/
```

## Troubleshooting

### Browser won't launch
```bash
# Install browser dependencies
npx playwright install-deps chromium
```

### Timeout errors
```javascript
// Increase timeout
CONFIG.timeout = 60000;
```

### Element not found
```javascript
// Wait longer for element
await page.waitForSelector('selector', { timeout: 10000 });
```

### Network issues
```javascript
// Wait for network idle
await page.waitForLoadState('networkidle');
```

## Output Format

The skill produces:

1. **Console output** - Real-time pass/fail results
2. **Screenshots** - PNG files for each test step
3. **report.json** - Machine-readable test results

```json
{
  "passed": ["Homepage loads (desktop)", "Login form elements (desktop)"],
  "failed": [{"test": "Form validation", "error": "Timeout"}],
  "screenshots": [
    {"name": "01-homepage-desktop", "filepath": "./ui-test-screenshots/01-homepage-desktop-2026-01-08.png"}
  ],
  "startTime": "2026-01-08T20:00:00.000Z",
  "endTime": "2026-01-08T20:01:30.000Z"
}
```
