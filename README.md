# bzhstack

Personal, harness-agnostic AI setup: global instructions ([AGENTS.md](./AGENTS.md)) and 13 curated skills ([skills/](./skills)) designed to live in `~/.agents` and be symlinked into whichever coding harnesses are installed.

**Recommended harness: [T3 Code](https://github.com/pingdotgg/t3code)** — its orchestrator capabilities (`orchestrator_capabilities` and `delegate_task`) let `bridge-room` and `council` fan out work across Claude, GPT, and Gemini in parallel (including isolated worktrees in arena mode).

## What's inside

### [AGENTS.md](./AGENTS.md)
Global rules shared across repositories and harnesses:
- **GitHub & issue trackers** — use `gh` CLI, follow each repo's configured tracker (ask before creating issues when unspecified), and link issues created by `/to-tickets`.
- **Operating contract** — risk-scaled investigation, act-don't-stall defaults, behavior-first tests, and evidence-backed verification.
- **Sub-agents, writing & disk hygiene** — direct work by default, concise plain language, and strict cleanup of only task-created artifacts.

### Skills

#### Understand
| Skill | Invoke | Purpose |
| --- | --- | --- |
| [how](./skills/how/SKILL.md) | `/how` | Code walkthroughs, runtime flow, and ownership/layering questions. |
| [why](./skills/why/SKILL.md) | `/why` | Design rationale and history from git commits, PRs, and discussions—not guesswork from code. |

#### Plan & break down
| Skill | Invoke | Purpose |
| --- | --- | --- |
| [grill-me](./skills/grill-me/SKILL.md) | `/grill-me` or auto | Round-based design-tree interview to stress-test a plan until no assumptions remain. |
| [to-spec](./skills/to-spec/SKILL.md) | `/to-spec` | Synthesize the current conversation and repo context into a spec (no interview) and publish it to the project's tracker. |
| [to-tickets](./skills/to-tickets/SKILL.md) | `/to-tickets` | Break a spec or conversation into tracer-bullet vertical-slice tickets with blocking edges. |

#### Review & multi-model
| Skill | Invoke | Purpose |
| --- | --- | --- |
| [blast-radius](./skills/blast-radius/SKILL.md) | `/blast-radius` or auto | Find what a change could break outside the diff and prove the core safety fact by running real code. |
| [bridge-room](./skills/bridge-room/SKILL.md) | `/bridge-room` / `arena` or auto | Run Claude, GPT, and Gemini in parallel via T3 Code's delegate tools—**ask mode** for second opinions, **arena mode** to build competing candidates in worktrees, cross-judge, and graft the best parts. |
| [council](./skills/council/SKILL.md) | `council` | Multi-model review rounds with `/10` scores, fix cycles, and PR review summary tables. |

#### Clean up
| Skill | Invoke | Purpose |
| --- | --- | --- |
| [no-comments](./skills/no-comments/SKILL.md) | `/no-comments` or auto | Strip comments outside a strict keep list ([rules.md](./skills/no-comments/rules.md)) and flag/fix the workaround code they excuse. |
| [prune-tests](./skills/prune-tests/SKILL.md) | `/prune-tests` or auto | Remove tests that guard no real behavior. |
| [unslop-ui](./skills/unslop-ui/SKILL.md) | `/unslop-ui` | Cut AI tells from UI and chart copy (`--surface` also cleans up gradients, spacing, weights, and decorative motion). |

#### Ship
| Skill | Invoke | Purpose |
| --- | --- | --- |
| [file-pr](./skills/file-pr/SKILL.md) | `/file-pr` or auto | Open concise, problem-first PRs. Automatically gates on `blast-radius` for cross-cutting or shared-invariant changes. |
| [leaving-pr-comments](./skills/leaving-pr-comments/SKILL.md) | auto | Standardized disclosure prefix for agent-authored GitHub PR comments. |

### How the skills compose
- **Design → tickets:** `grill-me` → `to-spec` → `to-tickets`
- **Pre-PR safety check:** `file-pr` → `blast-radius` → `bridge-room` (ask mode)
- **Comment audit:** `no-comments` → `why` (to verify suspicious comments against git history before keeping or deleting)
- **GitHub comments:** `file-pr` → `leaving-pr-comments`

## Setup

Copy the contents (excluding `.git`) into `~/.agents` so local edits stay decoupled from the repo:

```bash
mkdir -p ~/.agents
rsync -a --exclude='.git' --exclude='.DS_Store' ./ ~/.agents/
```

Then symlink `~/.agents/AGENTS.md` and `~/.agents/skills` into each harness:

```bash
# Antigravity (agy CLI)
mkdir -p ~/.gemini/config
ln -sfn ~/.agents/AGENTS.md ~/.gemini/config/AGENTS.md
ln -sfn ~/.agents/skills ~/.gemini/config/skills

# ZCode
ln -sfn ~/.agents/AGENTS.md ~/.zcode/AGENTS.md
ln -sfn ~/.agents/skills ~/.zcode/skills

# Pi
mkdir -p ~/.pi/agent
ln -sfn ~/.agents/AGENTS.md ~/.pi/agent/AGENTS.md
ln -sfn ~/.agents/skills ~/.pi/agent/skills
```

> **T3 Code (Antigravity provider):** T3 Code runs Antigravity with its own `GEMINI_HOME` under `~/.t3/userdata/providers/antigravity/<id>/config/`. Its `skills` symlink already chains to `~/.gemini/config/skills`; add `ln -sfn ~/.agents/AGENTS.md "$GEMINI_HOME/config/AGENTS.md"` so sessions outside `~` also load your global `AGENTS.md`.

## Credits

Several skills are adapted from open-source collections:
- [cursor/plugins (`pstack`)](https://github.com/cursor/plugins) — `how`, `why`, `blast-radius`, `no-comments`, and `arena` (adapted into `bridge-room`)
- [mattpocock/skills](https://github.com/mattpocock/skills) — `grill-me`, `to-spec`, `to-tickets`
- [t3dotgg/tstack](https://github.com/t3dotgg/tstack) — `council`, `leaving-pr-comments`
