# Problems & Solutions — Weston Labs Knowledge Base

**Last Updated:** May 4, 2026 · **Status:** Living Document  
**Purpose:** Capture recurring issues, root causes, and fixes so future sessions learn from what we've solved

---

## Overview

This is our institutional memory of problems we've encountered and how we fixed them. When something breaks, check here first before debugging from scratch.

**Pattern:** Problem → Root Cause → Solution → Prevention

**Related Documentation:**
- **DOMAIN_DNS_SETUP.md** — Complete DNS configuration guide for heyweston.ai (Cloudflare + Railway + Proton)
- **DEPLOYMENT_BEST_PRACTICES.md** — Pre-deployment checklist
- **DEPLOYMENT_DIAGNOSTIC_2026-05-04.md** — Diagnostic document from May 4 deployment fixes

---

## Deployment & Railway Issues

### 1. Package-Lock.json Out of Sync (heyweston.ai 502 Error)

**Symptom:** Service returns 502 error at runtime. Build succeeded, but service crashes.

**Root Cause:**
- Added `serve` to package.json but didn't run `npm install`
- Railway uses `npm ci` which requires package-lock.json and package.json to match exactly
- Missing lockfile entry for `serve` → npm ci fails → build error

**Timeline:** May 4, 2026 · Service: heyweston.ai · Commits: a8a9e74, 9661d3b

**Solution:**
```bash
# After adding ANY npm package:
npm install              # Regenerates package-lock.json with all deps
git add package.json package-lock.json
git commit -m "Add: [package] to dependencies"
git push
```

**Why it works:** `npm install` resolves dependency versions and generates integrity hashes. `npm ci` on Railway uses these exact hashes.

**Prevention:**
- Always run `npm install` locally after editing package.json
- Never manually edit package.json without running npm install
- Verify package-lock.json is in git diff before committing
- Follow pre-deployment checklist (see DEPLOYMENT_BEST_PRACTICES.md)

**Related Issues:**
- TypeScript compilation errors (see #2 below)
- Missing dependencies in package.json

---

### 2. TypeScript Type Mismatch (Dashboard Build Failure)

**Symptom:** Railway build fails at TypeScript compilation. Error: "Property 'done' does not exist on type..."

**Root Cause:**
- Added `done` property to return type but forgot to update the `Issue` interface
- TypeScript interface said `priority: 'urgent' | 'high' | 'medium'` but we tried to use `'done'`
- Caught by `npm run build` locally (should have run it before pushing)

**Timeline:** May 4, 2026 · Service: internal.heyweston.ai · Component: TasksSection · Commit: d8aa0dc

**Solution:**
```typescript
// ❌ WRONG: Return type has 'done' but interface doesn't
export interface Issue {
  priority: 'urgent' | 'high' | 'medium'
}
export function parseOpenIssues(): { done: Issue[] } { }

// ✅ RIGHT: Update interface to match return type
export interface Issue {
  priority: 'urgent' | 'high' | 'medium' | 'done'
}
export function parseOpenIssues(): { done: Issue[] } { }
```

**Why it works:** TypeScript interfaces define the shape of data. Return types must match what interfaces allow.

**Prevention:**
- Always run `npm run build` locally before pushing
- Update interfaces AND return types together (not separately)
- Use pre-deployment checklist

**Related Issues:**
- Package-lock.json sync (see #1 above)
- Missing imports

---

### 3. Missing Dependency in start Script

**Symptom:** 502 error. Service crashes at startup saying "command not found: serve"

**Root Cause:**
- package.json has `"start": "npx serve -s public"` 
- But `serve` is not in dependencies
- Railway installs nothing → `npx serve` fails → 502

**Timeline:** May 4, 2026 · Service: heyweston.ai · Commit: a8a9e74

**Solution:**
```json
{
  "scripts": {
    "start": "serve -s public -l 3000"
  },
  "dependencies": {
    "serve": "^14.2.0"
  }
}
```

**Why it works:** npm installs packages in dependencies. Using them in scripts requires they exist locally.

**Prevention:**
- Any package used in scripts must be in dependencies
- Run `npm ci` locally to test what Railway will do
- Check build logs in Railway immediately (don't wait)

**Learning:** This is a variant of problem #1 (lockfile sync). The root cause is not running npm install.

---

## Dashboard & Component Issues

### 4. Collapsible UI Not Showing on First Load

**Symptom:** Done section appears but expand/collapse toggle doesn't work or state persists across refreshes.

**Prevention Strategy:**
- Use React `useState` for UI state (expand/collapse)
- Use git state for data persistence (markdown Done section)
- Hard refresh (Cmd+Shift+R) after deployment to clear browser cache

**Related to:**
- React component state management
- Browser caching after Railway deploy

---

## Git & GitHub Workflow

### 5. Stale Local Branch After Failed Push

**Symptom:** `git push` fails, then local changes are out of sync with remote.

**Solution:**
```bash
git status                    # Check what's unpushed
git pull origin main          # Get latest from GitHub
git add [files]
git commit -m "..."
git push                      # Try again
```

**Prevention:**
- Always `git pull` before making changes
- Push frequently (don't accumulate commits)
- If push fails, diagnose why immediately (don't ignore it)

---

## Memory & Context Management

### 6. Memory File Not Being Read on Session Start

**Symptom:** Session loads but missing context from previous days.

**Prevention:**
- Memory files live in `memory/YYYY-MM-DD.md` (append-only, never overwrite)
- MEMORY.md is curated long-term memory (read-only during flushes)
- Check HEARTBEAT.md for standing tasks
- On session start, read MEMORY.md + today's memory file

**Related:**
- Pre-compaction memory flush (append-only rule prevents data loss)
- Archive system for old months

---

## API & Integration Issues

### 7. AgentMail Send Endpoint Wrong

**Symptom:** Email send fails with 404. Error says endpoint not found.

**Root Cause:**
- Tried `/messages` endpoint for sending
- Correct endpoint is `/messages/send` (not just `/messages`)

**Timeline:** April 2026 · Tool: agentmail__send_message

**Solution:**
```bash
# ❌ WRONG
POST /messages

# ✅ RIGHT
POST /messages/send
```

**Prevention:**
- Document correct endpoints in TOOLS.md when discovered
- Test API endpoints with curl before using in code
- Check skill documentation (see SKILL.md for agentmail)

**Status:** Fixed and documented in TOOLS.md

---

## Railway & Environment Variables

### 8. Missing Environment Variables on Deploy

**Symptom:** Service deploys but app crashes because it can't find config values.

**Common Variables Missing:**
- `NEXT_PUBLIC_GA_ID` (GA4 tracking)
- `NEXT_PUBLIC_BREVO_API_KEY` (email service)
- `NEXT_PUBLIC_BREVO_LIST_ID` (newsletter list)
- `NODE_ENV=production`
- `PORT=3000`

**Solution:**
1. Go to Railway dashboard
2. Select service → Variables
3. Add missing env vars
4. Redeploy (Railway auto-redeploys on var changes)

**Prevention:**
- Document all required env vars in service README
- Check Railway variables before marking deploy complete
- Use `.env.example` file in repo listing all possible vars

**Related:** numstack.net calculator needs GA4 and Brevo vars to function

---

## TypeScript & Build Errors

### 9. Cannot Find Module Error

**Symptom:** Build fails: "Cannot find module 'react'" or similar

**Root Cause:**
- Package is used but not in dependencies
- Or node_modules is stale (npm ci needed)

**Solution:**
```bash
npm install package-name       # Add it
npm ci                         # Or refresh node_modules
npm run build
```

**Prevention:**
- Import only from packages in dependencies
- Run `npm ci` after cloning or if build fails mysteriously
- Use pre-deployment checklist

---

## Performance & Optimization

### 10. Dashboard Slow to Load Data

**Prevention Strategy:**
- API route should batch fetch all needed files
- Parser should be efficient (don't re-read files multiple times)
- Consider caching if needed (but keep simple for now)

---

## Security & Access

### 11. API Keys or Secrets Exposed in Logs

**Prevention:**
- Never console.log API keys
- Use environment variables for secrets
- Check git diff before pushing (catch secrets before they go to GitHub)
- If secret was exposed, rotate it immediately

**Weston Labs Secrets:**
- AgentMail API key
- Brevo API key
- Anthropic API key
- Railway deployment tokens

---

### 12. Static Site Not Accessible (Port Hardcoded)

**Symptom:** Service logs show "Accepting connections at http://localhost:8080" but site is unreachable from public URL. Hangs or times out.

**Root Cause:**
- Start script hardcoded port 3000: `serve -s public -l 3000`
- Railway runs services on port 8080 (not 3000)
- serve was listening on 3000 inside Railway container
- Railway edge router expects service on port 8080 → timeout

**Timeline:** May 4, 2026 · Service: heyweston.ai (static site) · Commits: a8a9e74, 89d653f

**Solution:**
```json
{
  "scripts": {
    "start": "serve -s public -l ${PORT:-3000}"
  }
}
```

**Why it works:** Environment variable `${PORT:-3000}` means:
- Use PORT env var if Railway sets it (Railway sets PORT=8080)
- Default to 3000 for local development
- serve now listens on the correct port

**Prevention:**
- Never hardcode ports in start scripts
- Always use environment variables: `${PORT:-3000}`
- Test locally with PORT env var: `PORT=8080 npm start`
- Check Railway logs for "Accepting connections at" to verify port
- Pre-deployment checklist should include: "Start script respects PORT env var"

**Related Issues:**
- Package-lock.json sync (see #1)
- Missing dependencies (see #3)

---

## Self-Detected Errors (From Maintenance System)

**Framework:** SELF_MAINTENANCE_FRAMEWORK.md  
**Detection:** Agents monitor continuously and log findings  
**Pattern Matching:** WESTON_ERROR_DETECTION.md

**Adding New Errors:**
1. Agent detects issue via self-check or operation
2. Diagnoses root cause using WESTON_ERROR_DETECTION.md workflows
3. Adds entry here with full context
4. Links to error detection category for prevention

**Tracking:** All self-detected errors tracked in memory/YYYY-MM-DD.md + escalated if critical

---

## Decision Log

### May 4, 2026: Created PROBLEMS_SOLUTIONS.md

**Why:** Capture lessons learned so we don't repeat debugging.

**Included Today's Lessons:**
1. Package-lock.json must sync with package.json (heyweston.ai 502)
2. TypeScript interfaces must match return types (dashboard TS2339)
3. Dependencies must be declared for packages used in scripts
4. Pre-deployment checklist prevents 90% of issues

**Future Use:** When something breaks, check here first.

**Integration:** Add to website + link from HEARTBEAT.md

---

## Template: Adding New Issues

When we solve a new problem, add it here:

```markdown
### N. [Problem Title]

**Symptom:** What the user sees

**Root Cause:** Why it happened

**Timeline:** Date · Service/Component · Commit (if applicable)

**Solution:**
[Code or steps]

**Why it works:** Explanation

**Prevention:** How to avoid next time

**Related Issues:** Link to similar problems
```

---

## Quick Reference: Most Common Issues

| Issue | Symptom | Fix | Prevention |
|-------|---------|-----|-----------|
| Lockfile sync | Build fails at npm ci | Run npm install locally | Always npm install before push |
| Type mismatch | TS2339 compilation error | Update interface to match | Run npm run build locally |
| Missing package | Service crashes on start | Add to dependencies | Use pre-deployment checklist |
| Hardcoded port | Service hangs from outside | Use `${PORT:-3000}` env var | Test with PORT=8080 locally |
| Env var missing | App crashes on deploy | Add to Railway Variables | Document all required vars |
| Stale cache | Old UI after deploy | Hard refresh (Cmd+Shift+R) | Browser cache clears naturally |

---

## Search Tips

**Problem:** Service returns 502  
→ Check: Package-lock.json sync, missing dependencies, env vars

**Problem:** Build fails with TS error  
→ Check: TypeScript interface updates, import statements

**Problem:** Feature doesn't work after deploy  
→ Check: Env vars, hard refresh, Railway build logs

---

**Status:** Living document · Updated continuously as we learn  
**Owner:** Weston + Jeff  
**Next Review:** May 11, 2026

---

### 13. Linear MCP "Method Not Found" Error

**Symptom:** Linear MCP API calls return `{"error": {"code": -32601, "message": "Method not found"}}`

**Root Cause:**
- Tried calling tool methods directly: `{"method": "linear_create_issue", "params": {...}}`
- Linear MCP requires `tools/call` wrapper for all operations
- RPC layer doesn't recognize direct method names

**Timeline:** May 6, 2026 · Tool: Linear MCP · Status: RESOLVED

**Solution (CORRECT RPC FORMAT):**
```json
{
  "jsonrpc": "2.0",
  "method": "tools/call",
  "params": {
    "name": "linear_create_issue",
    "arguments": {
      "companyId": "5a9f30f3-1969-456e-a101-757feafbca13",
      "title": "Issue Title",
      "description": "Description",
      "priority": 1,
      "estimate": 4
    }
  }
}
```

**Why it works:**
- `tools/call` is the MCP RPC method that invokes tools
- Tool name goes in `params.name`
- Tool arguments go in `params.arguments`
- This pattern works for all MCP tools (not just Linear)

**Available Linear Tools:**
- `linear_create_issue` — Create new issue
- `linear_update_issue` — Update existing issue
- `linear_add_comment` — Add comment to issue
- `linear_get_issue` — Fetch issue details
- `linear_list_open_issues` — List all open issues
- `linear_list_states` — List possible issue states
- `linear_get_document` — Fetch document

**Prevention:**
- When using MCP: always use `tools/call` wrapper
- Never call tool methods directly
- Document correct format in skill/context
- Test RPC format with curl before using in code

**Keywords:** linear-mcp, rpc-format, tools-call-wrapper, method-not-found

**Related Issues:**
- None known (this was a documentation/format issue)

