# Agent Self-Repair System

**Purpose:** Enable agents to self-diagnose and fix common issues without human intervention.

**Framework:** When an agent encounters an error, it should:
1. Recognize the symptom
2. Search PROBLEMS_SOLUTIONS.md for matching pattern
3. Apply the documented solution
4. Log the finding + solution to memory
5. Return to work

**Status:** ACTIVE (May 6, 2026)  
**Owner:** Weston (CEO) + Agents  
**Updated:** Continuously as new issues are discovered

---

## Quick Access: Symptom → Solution

### Build & Deployment Failures

| Symptom | Check | Solution |
|---------|-------|----------|
| 502 error after deploy | PROBLEMS_SOLUTIONS.md #1, #3, #8 | npm install, package-lock sync, env vars |
| Build fails at compilation | PROBLEMS_SOLUTIONS.md #2, #9 | TypeScript interfaces, imports |
| Service won't start | PROBLEMS_SOLUTIONS.md #3, #12 | Missing deps, port config |
| Service hangs from outside | PROBLEMS_SOLUTIONS.md #12 | Port hardcoded (use `${PORT:-3000}`) |

### API & Tool Failures

| Symptom | Check | Solution |
|---------|-------|----------|
| Linear API returns "Method not found" | PROBLEMS_SOLUTIONS.md #13 | Use `tools/call` wrapper in RPC |
| AgentMail send fails with 404 | PROBLEMS_SOLUTIONS.md #7 | Use `/messages/send` endpoint |
| Any MCP tool failing | PROBLEMS_SOLUTIONS.md #13 | Check if using correct `tools/call` format |

### Feature Issues

| Symptom | Check | Solution |
|---------|-------|----------|
| Feature doesn't work after deploy | PROBLEMS_SOLUTIONS.md #8 | Check env vars, hard refresh, check logs |
| Collapsible UI not working | PROBLEMS_SOLUTIONS.md #4 | React state issue or browser cache |
| Old UI showing after deploy | PROBLEMS_SOLUTIONS.md #4 | Hard refresh (Cmd+Shift+R) |

### Git & Code Issues

| Symptom | Check | Solution |
|---------|-------|----------|
| Push fails, local out of sync | PROBLEMS_SOLUTIONS.md #5 | `git pull`, fix conflict, push |
| TypeScript errors before push | PROBLEMS_SOLUTIONS.md #2, #9 | Run `npm run build`, update types |

---

## Self-Repair Workflow

**When an agent (including Weston) encounters an error:**

### Step 1: Capture the Error
```markdown
- What happened: [symptom]
- When: [timestamp]
- Context: [what I was doing]
- Full error message: [paste error]
```

### Step 2: Search PROBLEMS_SOLUTIONS.md
- Read the quick reference table above
- Look for matching symptom
- If found: go to Step 4 (apply solution)
- If NOT found: go to Step 3 (diagnose & document)

### Step 3: Diagnose Unknown Error
Use diagnostic frameworks:
- **Build errors:** WESTON_ERROR_DETECTION.md (section: Build)
- **API errors:** WESTON_ERROR_DETECTION.md (section: API/Integration)
- **Runtime errors:** WESTON_ERROR_DETECTION.md (section: Runtime)
- **Memory errors:** SELF_MAINTENANCE_FRAMEWORK.md (section: Memory Health)

### Step 4: Apply Known Solution
1. Read the PROBLEMS_SOLUTIONS.md entry (symptom matches)
2. Follow the "Solution" section
3. Why it works: Read explanation to understand the fix
4. Prevention: Document what to avoid next time

### Step 5: Verify & Log
1. Confirm the error is resolved
2. Log to memory: problem, root cause, solution, prevention
3. If this is a NEW error type:
   - Add to PROBLEMS_SOLUTIONS.md (follow template)
   - Commit to git
   - Escalate to board if critical

### Step 6: Return to Work
Resume task. The error is solved.

---

## Error Categories

### A. Deployment & Infrastructure

**File:** PROBLEMS_SOLUTIONS.md (Issues #1-3, #8, #12)

**Typical agents:** Weston, subagents building/deploying apps

**Entry points:**
- Railway build fails
- Service returns 502
- Service hangs/times out
- Environment variables missing

**Self-fix success rate:** 95% (most are config or dependency issues)

---

### B. API & Integration Issues

**File:** PROBLEMS_SOLUTIONS.md (Issues #7, #13)

**Typical agents:** Weston, API-based tasks, integrations

**Entry points:**
- API call returns 404
- API returns "method not found"
- Endpoint doesn't exist
- RPC format wrong

**Self-fix success rate:** 90% (usually documentation/format)

---

### C. Code & Type Errors

**File:** PROBLEMS_SOLUTIONS.md (Issues #2, #9)

**Typical agents:** Developers, TypeScript users

**Entry points:**
- TypeScript compilation fails
- Property doesn't exist error
- Cannot find module
- Import not found

**Self-fix success rate:** 85% (requires understanding the type system)

---

### D. Git & Repository Issues

**File:** PROBLEMS_SOLUTIONS.md (Issue #5)

**Typical agents:** Any agent pushing code

**Entry points:**
- Push fails
- Merge conflicts
- Local branch out of sync

**Self-fix success rate:** 80% (git can be complex for new users)

---

### E. Memory & Context Management

**File:** SELF_MAINTENANCE_FRAMEWORK.md, HEARTBEAT.md

**Typical agents:** All agents (especially Weston)

**Entry points:**
- Can't find previous context
- Memory file empty
- Missing daily notes
- MEMORY.md not updating

**Self-fix success rate:** 90% (structured, follow template)

---

## When to Escalate

**Escalate to Jeff (board) if:**

1. **Error not in PROBLEMS_SOLUTIONS.md** + diagnostic doesn't help
2. **Error recurs after solution applied** (might be deeper issue)
3. **Error affects critical system** (security, data loss risk)
4. **Error blocks multiple tasks** (needs immediate attention)
5. **Solution requires permissions you don't have** (can't fix myself)

**Escalation format:**
```markdown
**Error:** [symptom]
**Diagnosis:** [what I tried]
**Evidence:** [error logs/timestamps]
**Impact:** [what can't I do?]
**Blockers:** [why I can't fix it]
```

---

## Adding New Problems to the Knowledge Base

**When you solve a NEW error type:**

1. **Don't stay silent** — document it
2. Go to PROBLEMS_SOLUTIONS.md
3. Add new section with template:
   ```markdown
   ### N. [Problem Title]
   
   **Symptom:** What the user sees
   **Root Cause:** Why it happened
   **Timeline:** Date · Component
   **Solution:** [code/steps]
   **Why it works:** Explanation
   **Prevention:** How to avoid
   **Related Issues:** Links
   ```
4. Commit to git: `"Add PROBLEMS_SOLUTIONS.md: [problem title]"`
5. Update this file if new category
6. Next agent will learn from your fix

---

## Agents & Self-Repair Authority

**Who can self-repair:**
- Weston (CEO) — Can fix anything, documents learning
- Subagents — Can fix issues in their task scope
- Oliver (persistent agent) — Can self-repair, documents findings
- Future agents — Follow this framework when deployed

**Who should NOT self-repair:**
- Anything requiring elevated permissions
- Security-critical systems without approval
- Changes that affect multiple systems
- Issues outside documented prevention

---

## Success Stories

### May 4, 2026: Weston Diagnosed 502 Error Chain

**Error sequence:**
1. Deploy heyweston.ai → 502
2. Checked build logs → package-lock.json out of sync
3. Ran `npm install` locally
4. Re-pushed → all good

**Learning:** Added PROBLEMS_SOLUTIONS.md entry #1 + #3  
**Future prevention:** Pre-deployment checklist (see DEPLOYMENT_BEST_PRACTICES.md)

---

### May 6, 2026: Weston Fixed Linear MCP "Method Not Found"

**Error:** Linear RPC calls return -32601 error  
**Investigation:** Tried multiple RPC formats  
**Discovery:** Needed `tools/call` wrapper (not direct method call)  
**Fix:** 10-line curl test, confirmed working  
**Learning:** Added PROBLEMS_SOLUTIONS.md entry #13 + updated TOOLS.md

---

## Connected Framework Files

**Use together:**

1. **PROBLEMS_SOLUTIONS.md** — Known issues database
2. **WESTON_ERROR_DETECTION.md** — Diagnostic procedures
3. **SELF_MAINTENANCE_FRAMEWORK.md** — Health checks (Weston-specific)
4. **DEPLOYMENT_BEST_PRACTICES.md** — Prevent issues before they happen
5. **OLIVER_SELF_MAINTENANCE_FRAMEWORK.md** — Specific to Oliver agent

---

## Monthly Audit

**First of every month (1st @ 1pm PDT):**

1. Review all memory files from past month
2. Extract new problems encountered
3. Check if in PROBLEMS_SOLUTIONS.md
4. If NEW: add entry with full context
5. Link to issue tracking (Linear)

**Owner:** Weston  
**Escalation:** If 5+ new error types in a month, flag to board for systemic review

---

## Living Document Policy

This file is NOT read-only. Update it when:
- New problem added to PROBLEMS_SOLUTIONS.md
- New error category identified
- Success story worth documenting
- Self-repair workflow improved

**When updating:**
- Preserve all existing entries
- Add to appropriate section
- Update modification date at top
- Commit to git with context

---

**Status:** ACTIVE · Last Updated: May 6, 2026 · Maintained by: Weston + Agents

