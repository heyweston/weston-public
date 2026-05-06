# Weston Guides — Open-Source Knowledge for AI Agents & Developers

A growing collection of guides, troubleshooting frameworks, and best practices for building AI agents, managing deployments, and solving common problems.

**Who this is for:**
- AI agents learning to self-repair
- Developers building with OpenClaw
- Teams automating with AI
- Anyone troubleshooting common deployment/infrastructure issues

**What's inside:**

## 📚 Core Guides

- **[Agent Self-Repair System](./docs/AGENT_SELF_REPAIR_SYSTEM.md)** — Agents self-diagnosing and fixing common errors without human help
- **[Problems & Solutions](./docs/PROBLEMS_SOLUTIONS.md)** — 13+ documented issues, root causes, and verified fixes
- **[Deployment Best Practices](./docs/DEPLOYMENT_BEST_PRACTICES.md)** — How to avoid common deployment failures

## 🚀 Getting Started

**If you're an AI agent:**
1. Read `AGENT_SELF_REPAIR_SYSTEM.md` (your playbook)
2. When stuck, search `PROBLEMS_SOLUTIONS.md` by symptom
3. Follow the solution documented
4. Log what you learned

**If you're a developer:**
1. Check `DEPLOYMENT_BEST_PRACTICES.md` before going live
2. Add new problems to `PROBLEMS_SOLUTIONS.md` as you solve them
3. Reference this repo in your team's docs

## 📋 Quick Reference

### Common Issues (Search Here First)

| Problem | Search Term | Guide |
|---------|-------------|-------|
| Service returns 502 | "package-lock.json" | PROBLEMS_SOLUTIONS.md |
| Build fails with TypeScript error | "TS2339" | PROBLEMS_SOLUTIONS.md |
| API returns "Method not found" | "RPC format" | PROBLEMS_SOLUTIONS.md |
| Service won't start | "hardcoded port" | PROBLEMS_SOLUTIONS.md |
| Feature broken after deploy | "env vars" | PROBLEMS_SOLUTIONS.md |

## 🛠️ How to Contribute

Found a problem we haven't documented? Help the next person!

1. **Add to PROBLEMS_SOLUTIONS.md**
   - Include: Symptom, Root Cause, Solution, Why It Works, Prevention
   - Follow the template in the file
   - Commit with: `"Add PROBLEMS_SOLUTIONS.md: [problem title]"`

2. **Create a new guide**
   - Use markdown + clear headings
   - Include examples/code when relevant
   - Commit with: `"Add guide: [title]"`

3. **Improve existing docs**
   - Clarifications always welcome
   - Add missing sections
   - Commit with: `"Improve [filename]: [what changed]"`

## 📈 Growth Plan

This repo starts focused on troubleshooting but will grow to include:
- Agent architecture patterns
- Deployment checklists
- Security guidelines
- Performance tuning
- Integration examples
- Team workflows

## 📄 License

Public domain — use freely, share widely, improve constantly.

---

**Status:** Active · Last Updated: May 6, 2026  
**Maintained by:** Weston Labs  
**Questions?** Open an issue or start a discussion

