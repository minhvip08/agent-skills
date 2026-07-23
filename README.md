# Agent Skills

**Production-grade engineering skills for AI coding agents — tuned for Java/Kotlin backend work.**

Skills encode the workflows, quality gates, and best practices that senior engineers use when building software. These ones are packaged so AI agents follow them consistently across every phase of development.

<a href="https://trendshift.io/repositories/25200" target="_blank"><img src="https://trendshift.io/api/badge/repositories/25200" alt="addyosmani%2Fagent-skills | Trendshift" style="width: 250px; height: 55px;" width="250" height="55"/></a>

![Addy's Agent Skills](https://addyosmani.com/assets/images/addys-agent-skills.jpg)

```
  DEFINE          PLAN           BUILD          REVIEW
 ┌──────┐      ┌──────┐      ┌──────┐      ┌──────┐
 │ Idea │ ───▶ │ Spec │ ───▶ │ Code │ ───▶ │  QA  │
 │      │      │  PRD │      │ Impl │      │ Gate │
 └──────┘      └──────┘      └──────┘      └──────┘
  /spec          /plan          /build        /review
```

---

## Commands

Slash commands that map to the development lifecycle. Each one activates the right skills automatically.

| What you're doing | Command | Key principle |
|-------------------|---------|---------------|
| Define what to build | `/spec` | Spec before code |
| Plan how to build it | `/plan` | Small, atomic tasks |
| Build incrementally | `/build` | One slice at a time |
| Prove it works | `/test` | Tests are proof |
| Review before merge | `/review` | Improve code health |
| Simplify the code | `/code-simplify` | Clarity over cleverness |
| Ship to production | `/ship` | Faster is safer |

Want fewer manual steps once the spec exists? **`/build auto`** generates the plan and implements every task in a single approved pass — you approve the plan once, then it runs autonomously. It removes the human stepping *between* tasks, not the verification: every task is still tested and committed individually, and it pauses on failures or risky steps.

Skills also activate automatically based on what you're doing — touching auth, input handling, or external calls triggers `security-and-hardening`; a suspected regression triggers `performance-optimization`; and so on.

Every generated artifact for a task (spec, plan, task list) lives together under `tasks/<YYYY-MM-DD>-<kebab-case-task-name>/`, so multiple tasks don't overwrite each other's files.

---

## Quick Start

**Claude Code (recommended):**

```
/plugin marketplace add addyosmani/agent-skills
/plugin install agent-skills@addy-agent-skills
```

**Local / development:**

```bash
git clone https://github.com/addyosmani/agent-skills.git
claude --plugin-dir /path/to/agent-skills
```

**Any other agent:** skills are plain Markdown files under `skills/<name>/SKILL.md` — point your agent's system prompt or instruction-file mechanism at the ones you want. The `commands/` directory packages the same slash commands for CLIs that read TOML-based command definitions (e.g. Antigravity).

---

## All 9 Skills

Each skill is a structured workflow with steps, verification gates, and anti-rationalization tables. Code examples throughout are Java/Kotlin.

### Define - Clarify what to build

| Skill | What It Does | Use When |
|-------|-------------|----------|
| [interview-me](skills/interview-me/SKILL.md) | One-question-at-a-time interview that extracts what the user actually wants instead of what they think they should want, until ~95% confidence | The ask is underspecified, or the user invokes "interview me" / "grill me" |
| [spec-driven-development](skills/spec-driven-development/SKILL.md) | Write a spec covering objectives, commands, structure, code style, testing, and boundaries before any code | Starting a new project, feature, or significant change |

### Plan - Break it down

| Skill | What It Does | Use When |
|-------|-------------|----------|
| [planning-and-task-breakdown](skills/planning-and-task-breakdown/SKILL.md) | Decompose specs into small, verifiable tasks with acceptance criteria and dependency ordering | You have a spec and need implementable units |

### Build - Write the code

| Skill | What It Does | Use When |
|-------|-------------|----------|
| [incremental-implementation](skills/incremental-implementation/SKILL.md) | Thin vertical slices - implement, test, verify, commit. Feature flags, safe defaults, rollback-friendly changes | Any change touching more than one file |
| [context-engineering](skills/context-engineering/SKILL.md) | Feed agents the right information at the right time - rules files, context packing, confusion management | Starting a session, switching tasks, or when output quality drops |

### Review - Quality gates before merge

| Skill | What It Does | Use When |
|-------|-------------|----------|
| [code-review-and-quality](skills/code-review-and-quality/SKILL.md) | Five-axis review, change sizing, severity labels, dead-code hygiene | Before merging any change |
| [code-simplification](skills/code-simplification/SKILL.md) | Chesterton's Fence, reduce complexity while preserving exact behavior | Code works but is harder to read or maintain than it should be |
| [security-and-hardening](skills/security-and-hardening/SKILL.md) | Threat modeling, OWASP prevention patterns, auth, secrets management, dependency auditing | Handling user input, auth, data storage, or external integrations |
| [performance-optimization](skills/performance-optimization/SKILL.md) | Measure-first approach - profiling workflow, common backend bottlenecks (N+1 queries, caching, GC/CPU) | Performance requirements exist or you suspect a regression |

---

## Agent Personas

Pre-configured specialist personas for targeted reviews:

| Agent | Role | Perspective |
|-------|------|-------------|
| [code-reviewer](agents/code-reviewer.md) | Senior Staff Engineer | Five-axis code review with "would a staff engineer approve this?" standard |
| [test-engineer](agents/test-engineer.md) | QA Specialist | Test strategy, coverage analysis, and the Prove-It pattern |
| [security-auditor](agents/security-auditor.md) | Security Engineer | Vulnerability detection, threat modeling, OWASP assessment |
| [web-performance-auditor](agents/web-performance-auditor.md) | Web Performance Engineer | Core Web Vitals audit with Quick/Deep modes; run it via `/webperf` if your project has a browser-facing surface |

`/ship` fans these personas out in parallel (`code-reviewer`, `security-auditor`, `test-engineer`) and synthesizes their reports into a single go/no-go decision.

---

## How Skills Work

Every skill follows a consistent anatomy:

```
┌─────────────────────────────────────────────────┐
│  SKILL.md                                       │
│                                                 │
│  ┌─ Frontmatter ─────────────────────────────┐  │
│  │ name: lowercase-hyphen-name               │  │
│  │ description: Guides agents through [task].│  │
│  │              Use when…                    │  │
│  └───────────────────────────────────────────┘  │                                                                                                
│  Overview         → What this skill does        │
│  When to Use      → Triggering conditions       │
│  Process          → Step-by-step workflow       │
│  Rationalizations → Excuses + rebuttals         │
│  Red Flags        → Signs something's wrong     │
│  Verification     → Evidence requirements       │
└─────────────────────────────────────────────────┘
```

**Key design choices:**

- **Process, not prose.** Skills are workflows agents follow, not reference docs they read. Each has steps, checkpoints, and exit criteria.
- **Anti-rationalization.** Every skill includes a table of common excuses agents use to skip steps (e.g., "I'll add tests later") with documented counter-arguments.
- **Verification is non-negotiable.** Every skill ends with evidence requirements - tests passing, build output, runtime data. "Seems right" is never sufficient.
- **Guideline-length.** Skills stay short enough to read in full — trimmed to the process and the anti-rationalization table, not padded into a reference manual.

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
├── agents/                            # 4 specialist personas
├── .claude/commands/                  # Slash commands (Claude Code)
├── commands/                          # Same slash commands, TOML format (other CLIs)
├── .claude-plugin/                    # Claude Code plugin + marketplace manifest
└── plugin.json                        # Plugin manifest
```

---

## Why Agent Skills?

AI coding agents default to the shortest path - which often means skipping specs, tests, and the practices that make software reliable. Agent Skills gives agents structured workflows that enforce the same discipline senior engineers bring to production code.

Each skill encodes hard-won engineering judgment: *when* to write a spec, *how* to slice work into safe increments, *how* to review, and *when* to simplify. These aren't generic prompts - they're the kind of opinionated, process-driven workflows that separate production-quality work from prototype-quality work.

Skills bake in best practices from Google's engineering culture — including concepts from [Software Engineering at Google](https://abseil.io/resources/swe-book) and Google's [engineering practices guide](https://google.github.io/eng-practices/). You'll find change sizing and review speed norms in code review, Chesterton's Fence in simplification, threat modeling (STRIDE) and the three-tier boundary system in security, and a measure-first discipline in performance work. These aren't abstract principles — they're embedded directly into the step-by-step workflows agents follow.

---

## Contributing

Skills should be **specific** (actionable steps, not vague advice), **verifiable** (clear exit criteria with evidence requirements), **battle-tested** (based on real workflows), and **minimal** (only what's needed to guide the agent).

See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines and the pre-flight checklist before proposing a new skill.

---

## Team

agent-skills is built and maintained by:

| | Name | GitHub | Role |
|---|------|--------|------|
| <img src="https://github.com/addyosmani.png?size=120" width="60" height="60" alt="Addy Osmani"> | **Addy Osmani** | [@addyosmani](https://github.com/addyosmani) | Creator |
| <img src="https://github.com/federicobartoli.png?size=120" width="60" height="60" alt="Federico Bartoli"> | **Federico Bartoli** | [@federicobartoli](https://github.com/federicobartoli) | Collaborator |
| <img src="https://github.com/nucliweb.png?size=120" width="60" height="60" alt="Joan León"> | **Joan León** | [@nucliweb](https://github.com/nucliweb) | Collaborator |

---

## License

MIT - use these skills in your projects, teams, and tools.
