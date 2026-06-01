---
name: audit
description: Audit a codebase for bugs, report findings BEFORE changing anything, then on approval fix, verify with a build/tests, and commit + push. Use when the user asks to "audit the codebase", "find bugs", "audit and fix", or run their standard audit-fix-verify-push loop.
---

# Bug Audit → Fix → Verify → Push

A disciplined, findings-first audit loop. The cardinal rule: **surface findings before
you start editing.** Do not disappear into a long silent file-reading phase — keep the
user in the loop and show results early.

## Workflow

1. **Scope fast, read smart.**
   - If the codebase is large, spawn parallel `Explore`/task agents to map it rather
     than reading every file serially on the main thread. Get to findings quickly.
   - Focus on likely bug categories for the stack at hand (e.g. navigation/lifecycle
     races, API pagination/read limits, edge-to-edge UI clipping, null/edge-case
     handling, resource leaks, lint/type errors, markdown link correctness).

2. **Report findings — then STOP.**
   - Present a concise **numbered list** of issues: file:line, what's wrong, severity,
     and the proposed fix in one line each.
   - **Do not change anything yet.** Wait for the user to confirm or pick which to fix.
   - If the user has pre-approved autonomous execution for this run, you may proceed —
     but still print the findings list first so it's visible.

3. **Fix.**
   - Apply the approved fixes. Match the surrounding code style. Don't introduce new
     features unless the user approves them.

4. **Verify — in the user's real environment.**
   - Run the actual build/test command (gradle, pytest, latexmk, npm, etc.).
   - On Windows, test in **PowerShell**, not just git bash. Watch for platform quirks
     (non-ASCII filenames/jobnames, path separators).
   - Iterate until the build is green. Report failures and the fix applied for each.
   - Before claiming "no data" / "can't be done", re-verify with an explicit
     count/command and show the output you based the conclusion on.

5. **Commit + push.**
   - Commit with a clear message, push to origin, rebase cleanly if needed.
   - Report the final repo state explicitly (branch, commit hash, push result).

## Principles

- **Findings before fixes.** Visible progress beats silent thoroughness.
- **Verify, don't assert.** Confident-but-wrong claims waste more time than a quick check.
- **Ship it.** The loop ends at a clean push, not a discussion — unless the user said otherwise.
