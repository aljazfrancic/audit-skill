---
name: audit
description: Audit a codebase for bugs, report verified findings BEFORE changing anything, then on approval fix, verify with a build/tests, and commit + push. Use when the user asks to "audit the codebase", "find bugs", "audit and fix", "check for bugs", or run their standard audit-fix-verify-push loop.
---

# Bug Audit → Fix → Verify → Push

A disciplined, findings-first audit loop. The cardinal rule: **surface findings before
you start editing.** Do not disappear into a long silent file-reading phase — keep the
user in the loop and show results early.

This is a **bug hunt**, not a general code review. Report defects: code that produces
wrong output, crashes, leaks, races, or silently corrupts state under some real input.
Do not report style preferences, speculative refactors, or "could be cleaner" — unless
the user asked for those too.

## Workflow

### 1. Scope fast, read smart

- Establish scope first: `git ls-files | head`, directory layout, stack, and size.
  If the user named a subsystem, stay inside it.
- If the codebase is large, fan out parallel `Explore` agents with **specific,
  non-overlapping scopes** (e.g. one per module/layer) rather than reading every file
  serially on the main thread. Ask each for candidate defects with file:line, not
  summaries. Get to findings quickly.
- Read the highest-risk code yourself: entry points, concurrency, I/O boundaries,
  error paths, anything recently changed (`git log --stat` is a cheap risk map).
- Focus on bug categories that fit the stack at hand, e.g.:
  - **Logic**: off-by-one, inverted conditions, wrong operator, unhandled enum/match arm,
    copy-paste errors where a sibling branch wasn't updated.
  - **Edge cases**: null/None/undefined, empty collections, zero/negative numbers,
    unicode and non-ASCII paths, timezone/DST, first-vs-subsequent run.
  - **Resources & lifecycle**: leaks (files, sockets, subscriptions, listeners),
    missing cleanup on error paths, navigation/lifecycle races, use-after-close.
  - **Concurrency**: shared mutable state, check-then-act races, missing awaits,
    fire-and-forget promises/futures whose errors vanish.
  - **Boundaries**: API pagination/read limits, partial reads/writes, retries without
    idempotency, timeouts, error responses treated as success.
  - **Security-adjacent correctness**: injection via string-built commands/queries/paths,
    secrets in logs, unvalidated external input reaching a trusting sink.
  - **Meta**: lint/type errors, broken markdown links, dead config that the code
    silently ignores.

### 2. Verify each finding before reporting it

A false positive costs the user more trust than a missed nit. Before a finding goes
in the list:

- **Re-read the actual code path** (not just the grep hit). Confirm the callers,
  guards, and types make the bug reachable.
- **State the failure scenario concretely**: what input or state leads to what wrong
  behavior. If you can't articulate that, it's not a finding — drop it or mark it
  explicitly as *unconfirmed*.
- Check it isn't already handled elsewhere (wrapper, caller-side guard, framework
  behavior) and isn't intentional (comments, tests that pin the behavior, git blame).

### 3. Report findings — then STOP

- Present a concise **numbered list, most severe first**. One finding per line-item:

  ```
  N. [severity] file:line — what's wrong.
     Failure: concrete input/state → wrong outcome.
     Fix: one-line proposed fix.
  ```

- Severity: **critical** (data loss, crash on common path, security), **high** (wrong
  behavior on realistic input), **medium** (edge-case bug, leak), **low** (latent
  hazard, meta issues).
- If the audit found nothing, say so plainly and show what you checked — do not
  invent findings to justify the run.
- **Do not change anything yet.** Wait for the user to confirm or pick which to fix.
- If the user has pre-approved autonomous execution for this run, you may proceed —
  but still print the findings list first so it's visible, and fix in severity order.

### 4. Fix

- **Baseline first**: run the build/tests once *before* editing and note pre-existing
  failures, so they aren't later blamed on your changes.
- Apply only the approved fixes, as **minimal diffs** that match the surrounding code
  style. No drive-by refactors, no new features unless the user approves them.
- If a fix turns out to be riskier or bigger than proposed, stop and say so before
  proceeding — don't silently expand scope.
- Where the project has tests, add or extend one that would have caught the bug,
  when that's cheap and idiomatic for the repo.

### 5. Verify — in the user's real environment

- Detect and run the project's actual build/test command (gradle, pytest, cargo,
  latexmk, npm/pnpm, make, …) — the same one CI or the user runs, not a substitute.
- On Windows, test in **PowerShell**, not just git bash. Watch for platform quirks
  (non-ASCII filenames/jobnames, path separators, line endings).
- Iterate until the result matches the baseline or better. Report each failure and
  the fix applied for it. **Never claim green without showing the passing output.**
- Before claiming "no data" / "can't be done", re-verify with an explicit
  count/command and show the output you based the conclusion on.

### 6. Commit + push

- **Branch safety**: if the working tree had uncommitted changes before the audit,
  or you're on a protected/default branch and the user hasn't already blessed pushing
  to it, ask before committing. Never force-push.
- Commit only the audit fixes (no unrelated files), with a message that says what was
  fixed and why. Group related fixes; don't jam ten unrelated bugs into one opaque commit.
- Pull/rebase cleanly if the remote moved, then push to origin.
- Report the final repo state explicitly: branch, commit hash(es), push result, and
  which findings were fixed vs. deferred.

## Principles

- **Findings before fixes.** Visible progress beats silent thoroughness.
- **Verify, don't assert.** A finding you can't demonstrate and a build you didn't run
  are both just guesses. Confident-but-wrong claims waste more time than a quick check.
- **Severity order, minimal diffs.** Fix what matters most, change only what the fix needs.
- **Ship it.** The loop ends at a clean push, not a discussion — unless the user said otherwise.
