# claude-audit-skill

A [Claude Code](https://claude.com/claude-code) skill that runs a disciplined
**audit → fix → verify → push** loop on a codebase.

Its one rule: **surface findings before touching anything.** No long silent
file-reading phase — you see a numbered list of verified issues first, approve what
to fix, and only then does it edit, verify against a real build/test, and commit + push.

## What it does

When you type `/audit` (or ask Claude to "audit the codebase for bugs"), it:

1. **Scopes fast** — fans out parallel exploration on large codebases instead of reading serially, and prioritizes high-risk code (entry points, concurrency, I/O, recent changes).
2. **Verifies findings before reporting** — each candidate bug is re-checked against the real code path with a concrete failure scenario, so you're not approving false positives.
3. **Reports findings, then stops** — a numbered list ranked by severity (file:line, failure scenario, proposed fix) for you to approve.
4. **Fixes** the approved issues as minimal diffs matching existing code style — after taking a test baseline, so pre-existing failures aren't confused with new ones.
5. **Verifies** with the project's real build/test command (gradle, pytest, cargo, latexmk, npm, …) in your actual shell, showing the passing output.
6. **Commits + pushes** safely (no force-push, asks before touching a dirty tree or protected branch), reporting the final repo state and which findings were fixed vs. deferred.

## Install

Copy the `audit` folder into your Claude Code skills directory.

**Global (all projects):**

```bash
# macOS / Linux
cp -r audit ~/.claude/skills/

# Windows (PowerShell)
Copy-Item -Recurse audit "$env:USERPROFILE\.claude\skills\"
```

**Per-project:** copy `audit` into `<your-repo>/.claude/skills/` instead.

Then run `/audit` in Claude Code. Confirm it loaded with `/skills`.

## License

MIT — see [LICENSE](LICENSE).
