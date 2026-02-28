---
name: code-review-expert
description: "Expert code review of current git changes with a senior engineer lens. Detects SOLID violations, security risks, and proposes actionable improvements."
---

# Code Review Expert

## Overview

Perform a structured review of the current git changes with focus on SOLID, architecture, removal candidates, and security risks. Default to review-only output unless the user asks to implement changes.

## Severity Levels

| Level | Name | Description | Action |
|-------|------|-------------|--------|
| **P0** | Critical | Security vulnerability, data loss risk, correctness bug | Must block merge |
| **P1** | High | Logic error, significant SOLID violation, performance regression | Should fix before merge |
| **P2** | Medium | Code smell, maintainability concern, minor SOLID violation | Fix in this PR or create follow-up |
| **P3** | Low | Style, naming, minor suggestion | Optional improvement |

## Scope Constraint: Diff-First, Legacy-Separate, Legacy-Non-Blocking

> **STRICT RULE — must be followed on every review:**
>
> 1. **Diff-First**: The primary review MUST focus **only on the current code changes** (the diff). Do not mix legacy findings into the main review findings.
> 2. **Legacy-Separate**: Existing/surrounding legacy code MAY be read for context. Any issues found in legacy code MUST be placed in a **separate, dedicated section** titled `## Legacy Code Observations` that appears **after** the main findings.
> 3. **Legacy-Non-Blocking**: Issues identified in legacy code MUST NOT block the current PR merge. They CANNOT be rated **P0** or **P1** for the current PR. They are treated as informational suggestions or tech-debt prompts only, suitable for future refactoring.

---

## Workflow

### 1) Preflight context

- Use `git status -sb`, `git diff --stat`, and `git diff` to scope changes.
- If needed, use `rg` or `grep` to find related modules, usages, and contracts.
- Identify entry points, ownership boundaries, and critical paths (auth, payments, data writes, network).

**Edge cases:**
- **No changes**: If `git diff` is empty, inform user and ask if they want to review staged changes or a specific commit range.
- **Large diff (>500 lines)**: Summarize by file first, then review in batches by module/feature area.
- **Mixed concerns**: Group findings by logical feature, not just file order.

### 2) SOLID + architecture smells

- Load `references/solid-checklist.md` for specific prompts.
- Look for:
  - **SRP**: Overloaded modules with unrelated responsibilities.
  - **OCP**: Frequent edits to add behavior instead of extension points.
  - **LSP**: Subclasses that break expectations or require type checks.
  - **ISP**: Wide interfaces with unused methods.
  - **DIP**: High-level logic tied to low-level implementations.
- When you propose a refactor, explain *why* it improves cohesion/coupling and outline a minimal, safe split.
- If refactor is non-trivial, propose an incremental plan instead of a large rewrite.

### 3) Removal candidates + iteration plan

- Load `references/removal-plan.md` for template.
- Identify code that is unused, redundant, or feature-flagged off.
- Distinguish **safe delete now** vs **defer with plan**.
- Provide a follow-up plan with concrete steps and checkpoints (tests/metrics).

### 4) Security and reliability scan

- Load `references/security-checklist.md` for coverage.
- Check for:
  - XSS, injection (SQL/NoSQL/command), SSRF, path traversal
  - AuthZ/AuthN gaps, missing tenancy checks
  - Secret leakage or API keys in logs/env/files
  - Rate limits, unbounded loops, CPU/memory hotspots
  - Unsafe deserialization, weak crypto, insecure defaults
  - **Race conditions**: concurrent access, check-then-act, TOCTOU, missing locks
- Call out both **exploitability** and **impact**.

### 5) Code quality scan

- Load `references/code-quality-checklist.md` for coverage.
- Check for:
  - **Error handling**: swallowed exceptions, overly broad catch, missing error handling, async errors
  - **Performance**: N+1 queries, CPU-intensive ops in hot paths, missing cache, unbounded memory
  - **Boundary conditions**: null/undefined handling, empty collections, numeric boundaries, off-by-one
- Flag issues that may cause silent failures or production incidents.

### 6) Commit message review

- Retrieve commit messages for the current changes using `git log --oneline` or `git log` as appropriate.
- For each commit message, verify:
  - **Clarity**: The message clearly describes *what* was changed and *why*.
  - **Meaningfulness**: The message is not vague (e.g., "fix", "update", "WIP", "misc changes") and provides enough context for a reviewer or future developer.
  - **Format**: The subject line is concise (≤72 characters recommended), uses imperative mood (e.g., "Add feature" not "Added feature"), and does not trail off without explanation.
  - **Scope**: Where multiple logical changes exist, each commit ideally represents a single coherent unit of work.
- For any commit message that is unclear, vague, or unhelpful, **explicitly suggest a revised message** using this format:
  ```
  ❌ Original: <original message>
  ✅ Suggested: <improved message that explains what changed and why>
  ```
- Include commit message findings in the review output under a dedicated section (see output format below).

### 7) Output format

Structure your review as follows:

```markdown
## Code Review Summary

**Files reviewed**: X files, Y lines changed
**Overall assessment**: [APPROVE / REQUEST_CHANGES / COMMENT]

---

## Findings
<!-- Scope: current diff changes ONLY. Do NOT include legacy code issues here. -->

### P0 - Critical
(none or list)

### P1 - High
1. **[file:line]** Brief title
  - Description of issue
  - Suggested fix

### P2 - Medium
2. (continue numbering across sections)
  - ...

### P3 - Low
...

---

## Removal/Iteration Plan
(if applicable)

## Commit Message Review
(none or list of commits with suggested improvements)

## Additional Suggestions
(optional improvements, not blocking)

---

## Legacy Code Observations
<!-- Issues found in existing/surrounding code that was NOT part of the current diff.
     These are NON-BLOCKING — they MUST NOT be rated P0 or P1 and do not affect the
     merge decision for this PR. Treat them as informational tech-debt prompts only. -->
(none or list suggestions for future refactoring)
```

**Inline comments**: Use this format for file-specific findings:
```
::code-comment{file="path/to/file.ts" line="42" severity="P1"}
Description of the issue and suggested fix.
::
```

**Clean review**: If no issues found, explicitly state:
- What was checked
- Any areas not covered (e.g., "Did not verify database migrations")
- Residual risks or recommended follow-up tests

### 8) Next steps confirmation

After presenting findings, ask user how to proceed:

```markdown
---

## Next Steps

I found X issues (P0: _, P1: _, P2: _, P3: _).

**How would you like to proceed?**

1. **Fix all** - I'll implement all suggested fixes
2. **Fix P0/P1 only** - Address critical and high priority issues
3. **Fix specific items** - Tell me which issues to fix
4. **No changes** - Review complete, no implementation needed

Please choose an option or provide specific instructions.
```

**Important**: Do NOT implement any changes until user explicitly confirms. This is a review-first workflow.

## Resources

### references/

| File | Purpose |
|------|---------|
| `solid-checklist.md` | SOLID smell prompts and refactor heuristics |
| `security-checklist.md` | Web/app security and runtime risk checklist |
| `code-quality-checklist.md` | Error handling, performance, boundary conditions |
| `removal-plan.md` | Template for deletion candidates and follow-up plan |
