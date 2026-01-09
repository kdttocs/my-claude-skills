---
name: mac-health
description: Reviews Mac system health, security settings, and performance. Use when the user asks to check their Mac's health, review system settings, audit security, optimize performance, or diagnose Mac issues.
allowed-tools: Bash, Read
---

# Mac Health Check

Perform a comprehensive review of macOS system health, security, and performance. Provide a prioritized list of improvements.

## Execution Steps

Run the following checks in order, collecting results for the final report.

### 1. Critical Security Checks

**FileVault Disk Encryption:**
```bash
fdesetup status
```
- PASS: "FileVault is On"
- FAIL: "FileVault is Off" - Recommend enabling for data protection

**Firewall Status:**
```bash
/usr/libexec/ApplicationFirewall/socketfilterfw --getglobalstate
```
- PASS: "Firewall is enabled"
- FAIL: "Firewall is disabled" - Recommend enabling in System Settings > Network > Firewall

**Gatekeeper (App Security):**
```bash
spctl --status
```
- PASS: "assessments enabled"
- FAIL: "assessments disabled" - Security risk, recommend re-enabling

**Pending Software Updates:**
```bash
softwareupdate -l 2>&1
```
- PASS: "No new software available"
- FAIL: Updates listed - Recommend installing, especially security updates

**Time Machine Backup Status:**
```bash
tmutil latestbackup 2>&1 || echo "Time Machine not configured"
```
- Check backup recency (warn if >7 days old or not configured)

### 2. Performance & Storage Checks

**Disk Space:**
```bash
df -h / | tail -1
```
- WARN if <15% free space
- CRITICAL if <5% free space

**Disk Health (SMART Status):**
```bash
diskutil info / | grep -E "SMART Status|Solid State"
```
- PASS: "Verified"
- FAIL: Any other status - Potential disk failure

**Memory Pressure:**
```bash
vm_stat | head -10
memory_pressure 2>/dev/null || echo "memory_pressure not available"
```
- Check for high page outs indicating memory pressure

**Battery Health (MacBooks only):**
```bash
system_profiler SPPowerDataType 2>/dev/null | grep -E "Cycle Count|Condition|Full Charge Capacity|Maximum Capacity"
```
- Check cycle count and condition
- WARN if condition is not "Normal"

**Login Items (Startup Programs):**
```bash
osascript -e 'tell application "System Events" to get the name of every login item' 2>/dev/null
sfltool dumpbtm 2>/dev/null | grep -E "Name|Path" | head -30
```
- List items that slow down startup
- Flag excessive login items (>10)

### 3. System Integrity Checks

**System Integrity Protection (SIP):**
```bash
csrutil status
```
- PASS: "enabled"
- FAIL: "disabled" - Security risk unless intentionally disabled

**Secure Boot (Apple Silicon):**
```bash
system_profiler SPiBridgeDataType 2>/dev/null | grep -i "secure boot"
```

**Privacy Permissions Audit:**
```bash
tccutil list 2>/dev/null || sqlite3 ~/Library/Application\ Support/com.apple.TCC/TCC.db "SELECT client,service FROM access" 2>/dev/null | head -20
```
- List apps with sensitive permissions (camera, microphone, screen recording, accessibility)

### 4. Network & Connectivity

**DNS Configuration:**
```bash
scutil --dns | grep "nameserver" | head -5
```

**Wi-Fi Security:**
```bash
/System/Library/PrivateFrameworks/Apple80211.framework/Versions/Current/Resources/airport -I 2>/dev/null | grep -E "SSID|auth|link auth"
```

### 5. Maintenance Status

**Spotlight Index Status:**
```bash
mdutil -s / 2>/dev/null
```

**System Uptime:**
```bash
uptime
```
- Note: Occasional restarts are healthy for system maintenance

**Purgeable/Cleanable Space:**
```bash
diskutil info / | grep -E "Purgeable|Container Free"
```

**macOS Version:**
```bash
sw_vers
```

## Output Format

After running all checks, produce a report in this format:

### Mac Health Report

**Overall Status:** [Healthy / Needs Attention / Critical]

**System:** [macOS version] on [Hardware]

---

#### Critical Issues (Fix Immediately)
List any security or data protection issues that need immediate attention.

#### High Priority (Fix Soon)
Performance, storage, or security issues that should be addressed.

#### Medium Priority (Recommended)
Maintenance and optimization suggestions.

#### Low Priority (Optional)
Nice-to-have improvements.

---

**Summary Statistics:**
- Storage: X% used (Y GB free)
- Memory: X GB / Y GB
- Battery: X% health, Y cycles (if applicable)
- Last Backup: [date or "Not configured"]
- Uptime: X days

---

For each issue found, provide:
1. **What:** Brief description of the issue
2. **Why it matters:** Impact on security/performance
3. **How to fix:** Specific steps or commands

## Prioritization Criteria

**Critical:**
- FileVault disabled
- Firewall disabled
- Security updates pending
- Disk health failing
- SIP disabled (unless intentional)

**High:**
- <10% disk space
- No recent backup (>7 days)
- High memory pressure
- Battery condition degraded
- Excessive login items (>10)

**Medium:**
- <20% disk space
- Backup >3 days old
- Spotlight issues
- Outdated non-security software
- DNS not using secure resolver

**Low:**
- System hasn't restarted in >30 days
- Cache cleanup recommended
- Minor optimization suggestions
