---
name: pr-reviewer
description: "Review a PR diff and flag risks, run security checklist, verify AC coverage, and score risk level. Use this before merging any PR."
argument-hint: "<PR number>"
---

# PR Reviewer

Review a pull request and post the findings as a comment on the PR.

## Project Context
```yaml
!`cat drupal-claude.yml 2>/dev/null || echo "No drupal-claude.yml found"`
```

## Protected Files
```yaml
!`cat drupal-claude.yml 2>/dev/null | grep -A10 'protected_files'`
```

## High Risk Modules
```yaml
!`cat drupal-claude.yml 2>/dev/null | grep -A5 'high_risk_modules'`
```

## Current Branch
```
!`git branch --show-current 2>/dev/null`
```

## PR Diff
```
!`git diff main 2>/dev/null`
```

## Instructions

### Step 1 — Analyze the PR

Review the PR thoroughly and produce the following report:

**Summary:**
What does this PR do in plain English? (2-3 sentences max)

**Risk Score:** Low / Medium / High

Scoring guide:
- Low: config YAML only, new files only, additive changes, no existing logic touched
- Medium: modifies existing logic, touches hooks, changes service behaviour
- High: touches protected files, entity hooks, OAuth flows, install hooks, cross-module changes

**Risk Reasons:**
- List specific reasons for the risk score

**Checklist:**
- [ ] Follows conventional commit format with Jira key
- [ ] References Jira ticket with "Resolves {KEY}-X" and Jira link
- [ ] No protected files modified without approval
- [ ] New code has tests or a documented manual test plan
- [ ] Config YAML has no hand-written uuid or _core keys
- [ ] No \Drupal::service() calls inside classes
- [ ] Services use constructor injection
- [ ] Quality checker has been run
- [ ] config:status clean before commit (if config changed)

**Security Checklist:**
- [ ] No raw SQL without parameterization (use Database API)
- [ ] No unescaped user input in output (XSS prevention)
- [ ] No hardcoded credentials or API keys
- [ ] Routes have `_permission` or `_access` requirements
- [ ] Entity access checks present where needed
- [ ] File uploads restricted to safe extensions (if applicable)
- [ ] No use of `eval()`, `exec()`, or `shell_exec()` on user input

**AC Coverage Matrix:**
Compare the spec's acceptance criteria against actual tests in the PR:
| AC | Criterion | Test File | Status |
|---|---|---|---|
| AC-1 | ... | tests/playwright/... | Covered / Missing |

**Flags — Needs Human Review:**
List specific files or lines a human must review before merge.
If none, say "None — safe to merge after automated checks pass."

**What's Missing:**
- Any tests that should exist but don't
- Any documentation that should be updated
- Any follow-up Jira tickets to create

**Verdict:**
One of:
- Approved — safe to merge
- Approved with comments — merge after addressing notes
- Changes requested — do not merge until resolved

---

### Step 2 — Fix any blocking issues

If verdict is "Changes requested":
- Fix the issues in the codebase
- Commit and push the fixes
- Re-run quality checks and tests
- Re-run this skill until verdict is "Approved" or "Approved with comments"

---

### Step 3 — Post the review as a PR comment

**This step is mandatory. Always post the review to GitHub.**

Use GitHub MCP add_issue_comment to post the complete review:
- owner: from drupal-claude.yml `github.owner`
- repo: from drupal-claude.yml `github.repo`
- issue_number: the PR number
- body: the complete review formatted in markdown below

Format the comment body as:

## Code Review

**Risk Score:** {Low/Medium/High}

### Summary
{summary}

### Checklist
{checklist with pass/fail for each item}

### Security Checklist
{security checklist with pass/fail for each item}

### AC Coverage
{AC coverage matrix}

### Risk Reasons
{reasons}

### Flags — Needs Human Review
{flags or "None — safe to merge"}

### What's Missing
{missing items or "Nothing — PR is complete"}

### Verdict
{full verdict with explanation}

---

### Step 4 — Confirm to user

After posting, tell the user:
- "Review posted as a comment on PR #{number}"
- The verdict
- Any items that need to be addressed before merging

## Hard Rules
- Always post the review to GitHub — never skip Step 3
- Never use event: "APPROVE" — GitHub blocks self-approval, always use add_issue_comment
- Always fix "Changes requested" findings before posting the final review
- Never tell the user the review is done without confirming it was posted to GitHub
- Never add attribution stamps to review comments
- Never use gh CLI commands — always use GitHub MCP tools instead
