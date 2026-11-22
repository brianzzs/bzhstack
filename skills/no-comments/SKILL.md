---
name: no-comments
description: Aggressively delete code comments and flag the workaround code they excuse. Use when the user says "no comments", "comment sicko", asks to remove, strip, or clean up comments, or wants a comment pass over a diff, branch, or files.
---

# No Comments

Delete every comment in scope that is not on the keep list, flag the code those comments were apologizing for, then fix what can be fixed cheaply.

The rules live in `rules.md` next to this file. Read it before starting; it is the only definition of what survives.

## Scope

The files, paths, or diff the user named. Otherwise the current diff against the base branch (default `main`), including uncommitted changes. Stay inside it.

## Steps

1. **Run the pass.** If this harness has a `comment-sicko` sub-agent, spawn it with the scope so it judges with fresh eyes. Otherwise do the pass yourself, following `rules.md` exactly. Either way, the pass deletes comments only and returns the report described in `rules.md`.
2. **Audit the pass.** Check the resulting diff. Reject application-code edits, edits outside scope, deletions of comments a keep clause protects, and `MUST KILL` flags with a false reason. Look for scoped lint and type suppressions it missed. Restore a deleted comment only when you can name its exact keep clause and prove it holds; if in doubt, it stays deleted. If the pass broke a rule, revert it and rerun once with the failure named; if it fails again, stop and report it.
3. **Fix the cheap flags.** For each accepted `MUST KILL` that has an obvious small fix (delete a dead path, drop an unused parameter, rename to say what the comment said, use the real API, fix the code a suppression was hiding), make it. Fix root causes; never add symptom guards.
4. **Report the expensive flags.** For flags that need a real reshape or reach outside the scope, do not refactor. Write a short sketch of the fix and list it as open work.
5. **Constraint comments.** For a deleted "do not remove", "do not change wording", or "talk to X first" comment about our own code, offer the cheapest enforcement that makes the comment unnecessary: a type, a runtime check, a test, or a lint rule. Implement it only if the user approves; otherwise list the constraint as open.
6. **Verify.** Run the repository's relevant checks (build, type-check, lint, tests) on what changed.

## Report

Deletion count, comments restored and why, `MUST KILL` flags fixed, flags left open with their sketch, constraint offers, and verification results.
