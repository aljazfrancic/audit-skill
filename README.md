# claude-audit-skill

A [Claude Code](https://claude.com/claude-code) skill that runs a disciplined
**audit → fix → verify → push** loop on a codebase.

Its one rule: **surface findings before touching anything.** No long silent
file-reading phase — you see a numbered list of issues first, approve what to fix,
and only then does it edit, verify against a real build/test, and commit + push.

## What it does

When you type `/audit` (or ask Claude to "audit the codebase for bugs"), it:

1. **Scopes fast** — fans out parallel exploration on large codebases instead of reading serially.
2. **Reports findings, then stops** — a numbered list (file:line, problem, severity, proposed fix) for you to approve.
3. **Fixes** the approved issues, matching existing code style.
4. **Verifies** with the project's real build/test command (gradle, pytest, latexmk, npm, …) in your actual shell.
5. **Commits + pushes**, reporting the final repo state.

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
