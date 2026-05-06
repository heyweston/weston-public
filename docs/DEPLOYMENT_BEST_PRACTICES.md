# Deployment Best Practices — Weston Labs

**Last Updated:** May 4, 2026 · **Status:** Foundation Document  
**Scope:** All web services (heyweston.ai, internal.heyweston.ai, numstack.net)

---

## Overview

This document captures lessons learned from deployment failures and establishes a repeatable, rock-solid process for code changes and deployments.

**Why this matters:** Every failed deployment costs time, credibility, and potentially revenue. Prevention > firefighting.

---

## Pre-Deployment Checklist

Run this **before every `git push`**. Takes 3-5 minutes. Saves hours of debugging.

### 1. Verify Dependencies

```bash
# For npm projects:
npm ci          # Clean install (respects lockfile exactly)
npm audit       # Check for vulnerabilities
```

**Why:** `npm ci` catches missing/mismatched packages before Railway sees them. `npm audit` flags security issues early.

### 2. Run Build

```bash
# For Next.js / TypeScript:
npm run build   # Catches TS errors, missing imports, broken configs

# For static sites:
npm run build   # If applicable, or manual verification
```

**Why:** Railway builds are slow and immutable. Catch errors locally where iteration is fast.

**Common errors caught here:**
- TypeScript type mismatches (e.g., missing interface properties)
- Missing dependencies in package.json
- Import path errors
- Configuration problems

### 3. Test Locally (if possible)

```bash
npm start       # Start dev/production server
# Visit http://localhost:3000 (or configured port)
# Verify basic functionality
```

**Why:** Proves the app actually runs, not just compiles.

### 4. Review Changes

```bash
git diff        # See exactly what's changing
git status      # Verify no accidental files are staged
```

**Why:** Catches uncommitted changes, staged mistakes, or unintended modifications.

### 5. Verify package-lock.json Sync

```bash
# After adding ANY npm dependency:
npm install     # Regenerates package-lock.json
git add package-lock.json
```

**Why:** npm ci (used by Railway) requires lockfile and package.json to be in sync. Mismatch = build failure.

**Real example from May 4:**
- Added `serve` to package.json
- Forgot to run `npm install`
- Railway's `npm ci` failed: "Missing: serve@14.2.6 from lock file"
- Fixed by regenerating lockfile

---

## Dependency Management

### Adding a New Package

```bash
npm install package-name
# This updates both package.json AND package-lock.json
git add package.json package-lock.json
git commit -m "Add: package-name for [reason]"
```

**Never do this:**
```bash
# ❌ BAD: Manually edit package.json without running npm install
# ❌ BAD: Add to package.json but forget to commit package-lock.json
# ❌ BAD: Use `npm install` locally but expect `npm ci` to work on Railway
```

### Updating Packages

```bash
npm update package-name
# Review changes in package-lock.json
git add package.json package-lock.json
git commit -m "Update: package-name from X to Y"
```

### Auditing Security

```bash
npm audit
npm audit fix           # For non-breaking fixes
npm audit fix --force   # Use with caution; may break things
```

---

## TypeScript / Build Errors

### Type Errors

**Common:** Adding a property to a return type but not updating the interface.

```typescript
// ❌ WRONG: Return type says done exists, but Issue interface doesn't include it
export function parseOpenIssues(): { done: Issue[] } { }
export interface Issue { priority: 'urgent' | 'high' | 'medium' }

// ✅ RIGHT: Both updated together
export interface Issue { priority: 'urgent' | 'high' | 'medium' | 'done' }
export function parseOpenIssues(): { done: Issue[] } { }
```

**Catch with:** `npm run build` (local) before pushing.

### Missing Imports

```typescript
// ❌ WRONG: Imported component not in dependencies
import { useState } from 'react'  // But react not in package.json

// ✅ RIGHT: Package is in dependencies
npm install react
```

**Catch with:** `npm run build` (local).

---

## Node.js Projects on Railway

### package.json Structure

```json
{
  "name": "project-name",
  "version": "1.0.0",
  "scripts": {
    "build": "...",           // Optional, if needed
    "start": "node server.js" // Required for Railway
  },
  "dependencies": {
    // List ALL runtime packages here
    "express": "^4.18.0"
  },
  "devDependencies": {
    // Build-time only
    "typescript": "^5.0.0"
  }
}
```

**Critical:**
- `start` script must exist and be production-ready
- All runtime dependencies in `dependencies` (not `devDependencies`)
- `package-lock.json` must always be committed

### Common Railway Issues

| Issue | Cause | Fix |
|-------|-------|-----|
| 502 Bad Gateway | Service crashes at runtime | Check logs; verify dependencies installed |
| Build fails at `npm ci` | Lockfile out of sync with package.json | Run `npm install` locally, commit package-lock.json |
| Port conflicts | Service trying to use wrong port | Verify `process.env.PORT` or hardcoded port; Railway sets PORT env var |
| Environment variables missing | Config not in Railway UI | Add to Railway Variables; redeploy |

---

## Git Workflow

### Before Pushing

1. ✅ Run pre-deployment checklist (above)
2. ✅ Verify build succeeds locally
3. ✅ Review git diff
4. ✅ Commit with clear message

### Commit Messages

```
Format: [Action]: [What] — [Why]

Examples:
✅ "Fix: Add serve dependency — npm ci requires lockfile sync"
✅ "Feature: Add collapsible Done section — Reduces dashboard clutter"
✅ "Update: Regenerate package-lock.json after npm install"

❌ "update stuff"
❌ "fix bugs"
❌ "changes"
```

### After Pushing

1. Railway auto-deploys (check project settings)
2. Monitor build logs in Railway UI
3. If build fails, fix immediately (don't wait)
4. Verify site works after deploy

---

## Debugging Failed Deployments

### Step 1: Check Railway Logs

```
Railway Dashboard → Service → Deployments → [Failed deploy] → Logs
```

Look for:
- Build errors (npm, TypeScript, etc.)
- Runtime errors (crashes, missing packages)
- Configuration issues (env vars, ports)

### Step 2: Reproduce Locally

```bash
git checkout <commit-hash>
npm ci
npm run build
npm start
```

Does it work locally? If yes, it's a Railway config issue (env vars, port, etc.).  
If no, it's a code issue.

### Step 3: Common Fixes

| Error | Fix |
|-------|-----|
| `npm ci` fails | Run `npm install` locally, commit package-lock.json, push |
| TypeScript error | Fix type annotations, run `npm run build` locally, push |
| Port conflict | Check Railway environment variables; set PORT if needed |
| Missing env var | Add to Railway Variables; redeploy |

---

## Service-Specific Guides

### heyweston.ai (Static Site)

**Stack:** Node.js + serve  
**Deployment:** Railway (automatic on main branch push)

**Pre-deployment:**
```bash
cd weston-labs
npm ci
npm run build  # If build exists
npm start      # Verify runs on localhost:3000
git diff       # Review changes
```

**Common issues:**
- Missing serve in package.json → 502 error
- Stale package-lock.json → Build fails at npm ci

### internal.heyweston.ai (Next.js Dashboard)

**Stack:** Next.js 14 + TypeScript + React  
**Deployment:** Railway (automatic on main branch push)

**Pre-deployment:**
```bash
cd dashboard
npm ci
npm run build  # Catches TypeScript errors
npm start      # Verify runs
git diff
```

**Common issues:**
- TypeScript type mismatches → Build fails
- Missing interface properties → Compilation error
- Stale node_modules → Run npm ci to fix

### numstack.net (Next.js Calculator)

**Stack:** Next.js 14 + TypeScript + Tailwind + GA4 + Brevo  
**Deployment:** Railway (automatic on main branch push)

**Pre-deployment:**
```bash
cd numstack
npm ci
npm run build
npm start
# Verify calculator works, email gate functions
```

**Environment variables needed:**
- `NEXT_PUBLIC_GA_ID` (GA4 Measurement ID)
- `NEXT_PUBLIC_BREVO_API_KEY` (Email service)
- `NEXT_PUBLIC_BREVO_LIST_ID` (Newsletter list)

---

## Prevention: Pre-Commit Hooks (Optional)

To automatically catch errors before pushing, add a pre-commit hook:

```bash
# File: .git/hooks/pre-commit
#!/bin/bash
set -e

echo "🔍 Running pre-deployment checks..."

npm ci --silent
npm run build --silent

echo "✅ Pre-commit checks passed"
```

Make executable:
```bash
chmod +x .git/hooks/pre-commit
```

Now `git commit` will fail if build breaks. Forces discipline.

---

## Monitoring After Deploy

### Basic Checks

1. **Is the service running?**
   ```bash
   curl -I https://heyweston.ai
   # Should return 200-399, not 502/503
   ```

2. **Are there runtime errors?**
   - Check Railway dashboard for crash logs
   - Review error tracking (Sentry, etc.) if configured

3. **Functional test**
   - Visit the site manually
   - Click key features
   - Verify expected behavior

### Alerts

Set up alerting for:
- Service 502/503 errors
- Build failures
- Crashes

---

## Decision Log

### May 4, 2026: Root Cause Analysis

**Problem:** Multiple deployment failures in one day  
**Pattern:** Missing dependencies + incomplete build checks  
**Decision:** Document complete best practices + pre-deployment checklist  
**Implementation:** This document + team discipline  
**Expected impact:** 90%+ reduction in preventable deploy failures

---

## Template: Pre-Deployment Checklist

Copy this before pushing:

```
[ ] npm ci (clean install succeeds)
[ ] npm audit (no critical vulnerabilities)
[ ] npm run build (build succeeds with no errors)
[ ] npm start (app runs locally)
[ ] git diff (review all changes)
[ ] package-lock.json committed (if deps changed)
[ ] Commit message is clear
[ ] Ready to git push
```

---

## Questions?

If a deployment fails:
1. Check the Railway logs (not guessing)
2. Reproduce locally (build + start)
3. Follow "Debugging Failed Deployments" above
4. Update this document if you find a new pattern

---

**Version:** 1.0  
**Next Review:** May 18, 2026  
**Owner:** Weston + Jeff
