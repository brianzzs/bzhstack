---
name: council
description: Trigger this if the user says 'council'.
---

# Council

Before an audit, address outstanding PR comments, fix known issues, and run
relevant checks. Give council the updated implementation; do not defer known
fixes until reviews return. If the user wants to discard the implementation
and explore new ideas, skip fixes to code being replaced.

Bring in three delegates to help with the user's request, usually for review
or feedback: Claude Opus 5.5, GPT 6.2 Sol, and Gemini 4 Neon, all at high
reasoning effort. If the user's instructions name other council members, use
those instead.

When this session runs inside T3 Code (its `orchestrator_capabilities` and
`delegate_task` tools are available), use `orchestrator_capabilities` to resolve the models, then
`delegate_task` to start one delegate per model in parallel. Run Opus on the
Claude provider (`claude-opus-5-5`, effort `high`), GPT 6.2 Sol on the
Codex provider (`gpt-6.2-sol`, `reasoningEffort` `high`), and Gemini 4 Neon
on the Antigravity provider (`gemini-4-neon`, `thinking_level` `high`).
Elsewhere, use the harness's subagent tools. If any member is unavailable,
report it instead of substituting.

Give each delegate the request and relevant context. For review or feedback,
keep their work read-only. Collect their responses and summarize the useful
findings, disagreements, and recommended next steps.

Do not wait idle. When a result arrives, check its findings against the code
at once and decide what to fix while the other delegates run. Do not edit
files while a delegate still reviews the same checkout. If the user's
instructions mark a delegate as non-blocking, do not wait for it; treat a
late result as feedback and do not record its score.

For a code review, ask each delegate for a score out of 10.
Count each full review pass as one round.
A fresh review after fixes starts a new round.

If the work ends in a pull request, put the results in the PR description.
Put this section before the model and harness line at the end:

```markdown
## Council review

Rounds: 2

| Delegate | Round 1 | Round 2 |
| --- | --- | --- |
| Claude Opus 5.5 | 7/10 | 9/10 |
| GPT 6.2 Sol | 8/10 | 10/10 |
| Gemini 4 Neon | 8/10 | 9/10 |
```

If a delegate gives no score, write `no score`. Do not invent one.
If the PR is open, edit the description. Do not post a comment for this.
If there is no PR, put the same results in the final reply only.

If you agree with their recommendations, make the changes and request a fresh
review after. Stop when every delegate scores at least 9/10, or after 3
rounds. The user can override either inline, e.g. "council, score 8, max 2".
At the round limit, stop and report the remaining findings instead of looping.
Out-of-scope findings do not block; list them as follow-ups.

Do not let repeated reviews result in scope creep. It is your job to fight to
keep scope limited.
