# Agent Skills

**Engineering skills for AI coding agents — tuned for Java/Kotlin backend work.**

Skills encode the workflows, quality gates, and best practices senior engineers use when building software, packaged so AI agents follow them consistently from spec to merge.

This is a trimmed, backend-focused fork of [addyosmani/agent-skills](https://github.com/addyosmani/agent-skills): fewer skills, shorter files, Java/Kotlin examples, and one folder per task for every generated artifact.

```
  DEFINE          PLAN           BUILD          REVIEW
 ┌──────┐      ┌──────┐      ┌──────┐      ┌──────┐
 │ Idea │ ───▶ │ Spec │ ───▶ │ Code │ ───▶ │  QA  │
 │      │      │ Plan │      │ Impl │      │ Gate │
 └──────┘      └──────┘      └──────┘      └──────┘
  /spec          /plan          /build        /review
```

---

## Quick Start

**Claude Code (plugin marketplace):**

```
/plugin marketplace add minhvip08/agent-skills
/plugin install agent-skills@minhvip08-agent-skills
```

The plugin manifest has no `version` field, so every commit pushed to `main` counts as a new version. Pull it with `/plugin marketplace update minhvip08-agent-skills`, or enable auto-update for this marketplace under `/plugin` → Marketplaces.

From a shell (e.g. when `/plugin` isn't available in your IDE extension):

```bash
claude plugin marketplace add minhvip08/agent-skills
claude plugin install agent-skills@minhvip08-agent-skills
```

**Local / development:**

```bash
git clone https://github.com/minhvip08/agent-skills.git
claude --plugin-dir /path/to/agent-skills
```

**Any other agent:** skills are plain Markdown files under `skills/<name>/SKILL.md` — point your agent's system prompt or instruction-file mechanism at the ones you want. The `commands/` directory packages the same slash commands as TOML for CLIs that read that format (e.g. Antigravity).

### Recommended companion tools

Optional, but they pair well with these skills — both cut how much context the agent burns, which is what `context-engineering` is about:

| Tool | What it does | Setup |
|------|-------------|-------|
| [codegraph](https://github.com/colbymchenry/codegraph) | Pre-indexed code graph exposed as an MCP server — symbols, call paths, and impact radius in one tool call instead of a chain of file reads. Supports Java and Kotlin. `context-engineering` and `code-review-and-quality` use it when present. | `curl -fsSL https://raw.githubusercontent.com/colbymchenry/codegraph/main/install.sh \| sh`, then `codegraph install` once and `codegraph init` in each project |
| [rtk](https://github.com/rtk-ai/rtk) | CLI proxy that compresses the output of common dev commands (git, test runners, builds) before it reaches the agent — typically 60–90% fewer tokens. | `curl -fsSL https://raw.githubusercontent.com/rtk-ai/rtk/refs/heads/master/install.sh \| sh` (or `brew install rtk`), then `rtk init -g` and restart Claude Code |

The skills don't require either tool — every rule that mentions a code index falls back to normal file reads when none is installed. rtk's hook only rewrites Bash commands; Claude Code's built-in Read/Grep tools bypass it.

---

## Commands

| What you're doing | Command | Key principle |
|-------------------|---------|---------------|
| Define what to build | `/spec` | Spec before code — and stop for approval |
| Plan how to build it | `/plan` | Small, verifiable tasks |
| Build incrementally | `/build` | One slice at a time |
| Review before merge | `/review` | Improve code health |
| Simplify the code | `/code-simplify` | Clarity over cleverness |

**`/build auto`** generates the plan if needed and implements every task in one approved pass — you approve the plan once, then it runs without stepping between tasks. Every task is still tested and committed individually, and it pauses on failures or risky steps.

Skills also activate on their own: each skill's `description` is always visible to the agent, so touching auth or input handling pulls in `security-and-hardening`, a suspected regression pulls in `performance-optimization`, and so on — no command needed.

### Where artifacts go

Every generated artifact for one task lives in one folder, so tasks never overwrite each other:

```
tasks/<YYYY-MM-DD>-<kebab-case-task-name>/
├── SPEC.md    # written by /spec
├── plan.md    # written by /plan (or /build if missing)
└── todo.md    # task checklist /build works through
```

`/plan` and `/build` find the most recently modified matching folder and ask if more than one is a plausible match.

---

## All 9 Skills

Each skill is a short, structured workflow with verification gates and an anti-rationalization table. Code examples are Java/Kotlin.

### Define — Clarify what to build

| Skill | What It Does | Use When |
|-------|-------------|----------|
| [interview-me](skills/interview-me/SKILL.md) | One question at a time, each with a guess attached, until ~95% confidence in what the user actually wants; stops the turn after an explicit yes | The ask is underspecified, or the user says "interview me" / "grill me" |
| [spec-driven-development](skills/spec-driven-development/SKILL.md) | Spec covering objective, commands, structure, code style, testing, and boundaries; stops for approval before any planning | Starting a new project, feature, or significant change |

### Plan — Break it down

| Skill | What It Does | Use When |
|-------|-------------|----------|
| [planning-and-task-breakdown](skills/planning-and-task-breakdown/SKILL.md) | Vertical slices, task sizing, acceptance criteria, checkpoints; owns the `plan.md` / `todo.md` layout | You have a spec and need implementable tasks |

### Build — Write the code

| Skill | What It Does | Use When |
|-------|-------------|----------|
| [incremental-implementation](skills/incremental-implementation/SKILL.md) | Implement → test → build → stage per slice; strict scope discipline, lint/format only changed files, commit as a separate step | Any change touching more than one file |
| [context-engineering](skills/context-engineering/SKILL.md) | Rules files, context packing, context budget (trim at ~75%), restartable session boundaries, confusion management | Starting a session, switching tasks, or when output quality drops |

### Review — Quality gates before merge

| Skill | What It Does | Use When |
|-------|-------------|----------|
| [code-review-and-quality](skills/code-review-and-quality/SKILL.md) | Five-axis review, change sizing, severity labels (Critical / Required / Consider / Nit), mutation check on tests | Before merging any change |
| [code-simplification](skills/code-simplification/SKILL.md) | Chesterton's Fence, one simplification at a time, behavior preserved exactly | Code works but is harder to read or maintain than it should be |
| [security-and-hardening](skills/security-and-hardening/SKILL.md) | STRIDE threat model, three-tier boundaries, SSRF, leaked-secret rotation, dependency and supply-chain rules, LLM features | Untrusted input, auth, sensitive data, dependencies, external calls |
| [performance-optimization](skills/performance-optimization/SKILL.md) | Measure-first workflow, backend bottleneck table (JFR, async-profiler), N+1, keyset pagination, bounded caches | Performance requirements exist or a regression is suspected |

---

## Agent Personas

| Agent | Role | Perspective |
|-------|------|-------------|
| [code-reviewer](agents/code-reviewer.md) | Senior Staff Engineer | Five-axis review using the same severity labels as `code-review-and-quality` |
| [test-engineer](agents/test-engineer.md) | QA Specialist | Test strategy, coverage analysis, and the Prove-It pattern |
| [security-auditor](agents/security-auditor.md) | Security Engineer | Vulnerability detection, threat modeling, OWASP assessment |

---

## How Skills Work

```
┌─────────────────────────────────────────────────┐
│  SKILL.md                                       │
│                                                 │
│  ┌─ Frontmatter ─────────────────────────────┐  │
│  │ name: lowercase-hyphen-name               │  │
│  │ description: [What it does]. Use when…    │  │
│  └───────────────────────────────────────────┘  │
│  Overview         → What this skill does        │
│  When to Use      → Triggering conditions       │
│  Process          → Step-by-step workflow       │
│  Rationalizations → Excuses + rebuttals         │
│  Red Flags        → Signs something's wrong     │
│  Verification     → Evidence requirements       │
└─────────────────────────────────────────────────┘
```

**Design choices:**

- **Process, not prose.** Skills are workflows with steps, checkpoints, and exit criteria — not reference docs.
- **Only what changes behavior.** A line stays only if it makes the agent act differently from its default. Generic knowledge the model already has (parameterize SQL, hash passwords) is a one-line rule, not a code sample.
- **No duplication.** Each rule lives in one skill; others reference it (dependency rules → `security-and-hardening`, scope rules → `incremental-implementation`, task files → `planning-and-task-breakdown`).
- **Short descriptions.** The `description` is loaded in every session, so it says what the skill does and when to use it — nothing more.
- **Human gates are real turns.** `interview-me` and `spec-driven-development` end the turn after confirmation instead of rolling straight into the next phase.
- **Verification is non-negotiable.** "Seems right" is never sufficient — tests, build output, measurements.

---

## Project Structure

```
agent-skills/
├── skills/                            # 9 skills (Java/Kotlin backend focused)
│   ├── interview-me/                  #   Define
│   ├── spec-driven-development/       #   Define
│   ├── planning-and-task-breakdown/   #   Plan
│   ├── incremental-implementation/    #   Build
│   ├── context-engineering/           #   Build
│   ├── code-review-and-quality/       #   Review
│   ├── code-simplification/           #   Review
│   ├── security-and-hardening/        #   Review
│   └── performance-optimization/      #   Review
├── agents/                            # 3 specialist personas
├── .claude/commands/                  # Slash commands (Claude Code)
├── commands/                          # Same commands, TOML format (other CLIs)
└── .claude-plugin/                    # Claude Code plugin + marketplace manifest
```

---

## Syncing with Upstream

Upstream changes are ported selectively rather than merged, since this fork removed or rewrote most skills:

```bash
git remote add upstream https://github.com/addyosmani/agent-skills.git   # once
git fetch upstream
git diff <last-synced-commit> upstream/main -- skills/ agents/ .claude/commands/
```

Port only what applies to the remaining skills and keeps them stack-neutral or Java/Kotlin.

---

## Contributing

Skills should be **specific** (actionable steps), **verifiable** (clear exit criteria), **battle-tested** (based on real workflows), and **minimal** (only what's needed to guide the agent).

See [CONTRIBUTING.md](CONTRIBUTING.md) for the pre-flight checklist before proposing a new skill.

---

## Credits

Based on [addyosmani/agent-skills](https://github.com/addyosmani/agent-skills) by [Addy Osmani](https://github.com/addyosmani), with collaborators [Federico Bartoli](https://github.com/federicobartoli) and [Joan León](https://github.com/nucliweb). Skills draw on [Software Engineering at Google](https://abseil.io/resources/swe-book) and Google's [engineering practices guide](https://google.github.io/eng-practices/).

## License

MIT — use these skills in your projects, teams, and tools.
