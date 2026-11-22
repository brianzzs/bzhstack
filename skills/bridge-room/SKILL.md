---
name: bridge-room
description: "Ask other models (Claude, GPT, Gemini) through T3 Code's delegate tools. Ask mode puts one question to each and merges the answers; arena mode has each build a candidate, then picks a base and grafts the best parts of the others. Use when the user says 'bridge-room' or 'arena', or when another skill asks for other models' opinions."
---

# Bridge room

Put other models on the same task through T3 Code's delegate tools, then merge what comes back. Different models catch different real bugs and take different shapes, so the value is in comparing them, not in any single answer.

## Before you start

This skill needs a session running inside T3 Code, where the `orchestrator_capabilities` and `delegate_task` tools are available. Without them, say that bridge-room is unavailable and stop; the caller carries on without it. Don't substitute same-model subagents.

Call `orchestrator_capabilities` once to resolve the models, then start one `delegate_task` per model, all in parallel. Default roster, all at high effort:

- Claude Opus 5.5: Claude provider, `claude-opus-5-5`, effort `high`.
- GPT 6.2 Sol: Codex provider, `gpt-6.2-sol`, `reasoningEffort` `high`.
- Gemini 4 Neon: Antigravity provider, `gemini-4-neon`, `thinking_level` `high`.

Use the models the user or the calling skill names instead, when it names any. Put settings in `target.options`. If a model is unavailable, report it instead of substituting, and continue with at least two. Don't poll a running delegate: follow the `next` its result gives you, and `task_cancel` any you no longer need.

Give every delegate the same prompt: the task, the relevant context and paths, and what to return. The prompt is the contract, so write it once.

## Ask mode (default)

For a question, a review, or a second opinion. One round, read-only.

1. Send each delegate the question and tell it not to edit files.
2. As each answer arrives, check its claims against the code. Don't wait idle for the rest.
3. Return, in this order: what the delegates agree on, where they disagree and which side the code supports, and anything only one of them raised that holds up. Separate real breakage from out-of-scope follow-ups and nits. Cite `file:line` where it applies. Mark claims you couldn't check as unverified.

## Arena mode

For a non-trivial artifact (a design, a module, a document) where one attempt could lock in the wrong shape. Use it when the user says "arena", or the caller asks for candidates rather than opinions.

1. **Frame.** State the artifact and write a rubric of 3 to 6 concrete, gradeable criteria for this task. The delegates see only the task; the rubric is for picking.
2. **Fan out.** Start each delegate with `isolation: "worktree"`, so each builds in its own checkout. Ask for the artifact plus a short rationale naming the alternatives it considered and rejected. If one produces nothing, continue with the rest and note the dropout.
3. **Cross-judge.** Once all candidates are in, start one read-only judge from a vendor other than this session's model (Fable counts as Claude). It scores each candidate against the rubric, criterion by criterion, and recommends a base.
4. **Pick a base.** Read every candidate end to end and score them yourself, criterion by criterion, then compare with the judge. Agreement confirms the pick. Disagreement means a bias or an ambiguous rubric, so read both rationales before deciding. Between two that feel tied, prefer the cleaner boundary or the smaller API.
5. **Graft.** Go through each losing candidate once for what is worth porting, usually one or two things each. Rework each graft into the base by hand so the result reads as one design; don't paste. If the candidates converge, ship the shared shape and say so. If they diverge wildly, the frame was under-specified: reframe and rerun instead of averaging.
6. **Verify.** Check the result like any other work: run the checks, the tests, or the code. If verification finds a problem, either the frame was wrong (rerun) or a candidate caught it and the graft missed it (go back to step 5).

Return the artifact with a short note: the base and why, the judge's verdict, each graft and the candidate it came from, what was rejected, any convergence or dropouts, and the verification result. The worktrees belong to T3 Code; leave them for the user to clean up.

## Good and bad examples

Ask mode (especially paired with `blast-radius` before opening a PR) is meant to catch cross-system bugs that pass CI because they live outside the diff.

### Prompts

- **Bad:** "Review this diff and tell me if anything looks wrong." Delegates stay inside the changed lines, confirm what tests already cover, and miss untouched callers, jobs, and state machines.
- **Good:** Name the changed invariant and the neighboring systems that touch the same state (global query filters, background sweepers, webhooks, concurrent handlers, client caches). Require `file:line`, a step-by-step failure path, and out-of-scope items separated from blockers.

### Findings

**Good — real breakage beyond the diff, with a concrete failure path:**
- **Global filter breaks an untouched background job:** Adding a soft-delete query filter hides deleted rows from an orphan-media sweeper. Assets older than 24h look unreferenced and get deleted from storage during the 7-day restoration window, breaking restore even though every test in the diff passes.
- **External async flow outlives a local guard:** Starting a Stripe Checkout records nothing that blocks entity deletion. An owner deletes the entity, then finishes paying in the open tab; the webhook's filtered `SingleAsync` throws 500 on every retry, billing a deleted entity with no UI left to cancel.
- **Wrong cross-feature state assumption:** Restore checks `IsActive`, missing that scheduled account deletion leaves `IsActive = true` while a deletion timestamp is set. Restoring re-attaches a visible entity, so the 30-day erasure job skips that owner indefinitely.
- **Client cache or stale-dialog race:** Navigating before removing a deleted item from cached lists leaves it clickable during a slow refetch; returning a name-mismatch error without refreshing the entity traps the user in the confirmation dialog after a concurrent rename.

**Bad — noise to cut, verify away, or downgrade to follow-ups:**
- **Vague warnings without a path:** "Global query filters might affect other queries" with no named caller, `file:line`, or reachable failure sequence.
- **Diff-only reassurance:** Stopping once auth, row locks, migration, and unit tests in the diff look clean, without checking background jobs, webhooks, or sibling handlers.
- **Scope creep as blockers:** Treating pre-existing or product-policy tradeoffs (previously issued signed URLs staying valid until expiry, unique-key reuse policy) as merge blockers instead of out-of-scope follow-ups.
- **Nits weighted like bugs:** Mixing minor copy tweaks, helper duplication, or low-impact edge cases (e.g. `U+FEFF` trimming differences between JS and .NET) in with data loss or billing breakage.
- **Unchecked claims:** Repeating a delegate's finding—especially a single-model find—without opening the cited `file:line` and confirming the failure actually happens.
