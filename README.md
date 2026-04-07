# harness-engineering

An on-demand skill that teaches AI coding agents (Claude Code, Cursor, Codex, CodeBuddy, Trae, Windsurf, Augment, Copilot, OpenClaw, OpenHarness, WorkBuddy, Manus, etc.) harness engineering best practices — the discipline of making agents reliable on complex, multi-step software tasks.

## What This Is

A skill your AI agent loads **only when it needs it** — for complex tasks spanning many files, requiring multi-step execution, or running across multiple sessions. Everyday tasks don't need it; complex ones do.

When loaded, your agent gets better at:

- **Reconnaissance** — understanding a codebase before changing it
- **Atomic changes** — one concern per commit, verified immediately
- **Context management** — not drowning in stale context or losing track of progress
- **Multi-session handoffs** — resuming work cleanly after a context window reset
- **Self-verification** — testing as a user, not just reading code
- **Failure recovery** — stopping, reverting, and retrying with a new hypothesis

## When to Load This Skill

Load it when the task:
- Touches **3+ files** or requires **5+ distinct steps**
- Involves an **unfamiliar or partially understood** codebase
- Requires **stability** (production system, shared codebase)
- Spans a **long session** where context can drift
- Is **too large for a single context window** (multi-session work)

**Don't** load it for quick fixes, single-file edits, or simple questions.

## Quick Start

### Option A: As an on-demand agent skill

Most AI coding agents support on-demand skills that load only when relevant. This is the **recommended** approach — it keeps your agent's context clean for everyday tasks and brings in the full methodology only when complexity demands it.

| Agent | Skill Directory | Setup |
|-------|----------------|-------|
| Claude Code | `.claude/skills/` | Place `SKILL.md` at `.claude/skills/harness-engineering/SKILL.md` |
| Cursor | `.cursor/skills/` | Place `SKILL.md` at `.cursor/skills/harness-engineering/SKILL.md` |
| CodeBuddy (Tencent) | `.codebuddy/skills/` | Place `SKILL.md` at `.codebuddy/skills/harness-engineering/SKILL.md` (project) or `~/.codebuddy/skills/harness-engineering/SKILL.md` (user) |
| WorkBuddy (Tencent) | `.workbuddy/skills/` | Place `SKILL.md` at `.workbuddy/skills/harness-engineering/SKILL.md` (project) or `~/.workbuddy/skills/harness-engineering/SKILL.md` (user) |
| OpenClaw | `~/.openclaw/workspace/skills/` | Place `SKILL.md` in skills directory |
| OpenHarness (`oh`) | `~/.openharness/skills/` | Place `SKILL.md` in skills directory — compatible with `.md` skill format |

### Option B: Reference from your agent instruction file

For agents that don't have a dedicated skill directory — or where you prefer a git submodule — add a **conditional reference** in your agent instruction file:

```bash
git submodule add https://github.com/rickyma/harness-engineering .harness
```

Then add this to your instruction file (`CLAUDE.md`, `.cursor/rules/*.md`, `AGENTS.md`, `.windsurfrules`, `.github/copilot-instructions.md`, etc.):

```markdown
When working on complex tasks (3+ files or 5+ steps), load and follow
the harness engineering methodology in .harness/SKILL.md.
For simple tasks, skip it.
```

This works with any agent that reads repo-local instruction files, including Codex, Trae, Windsurf, Augment, Copilot, Zed, Lingma, Comate, and others.

### Option C: Upload as knowledge

For agents that support knowledge uploads (Manus, custom GPTs, etc.), upload `SKILL.md` as a knowledge file with instructions to apply it only on complex tasks.

### What NOT to do

**Don't** copy `SKILL.md` into always-loaded instruction directories (`.cursor/rules/`, root `CLAUDE.md`, `.trae/rules/`, `.windsurfrules`, `.github/copilot-instructions.md`, etc.). At ~365 lines, it would consume context on every session — including simple tasks that don't need it. The whole point of harness engineering is disciplined context management; always-loading a 365-line methodology file contradicts that principle.

**Don't** paste `SKILL.md` content into your system prompt or custom instructions. Load it as a file/skill that the agent reads on demand.

## File Structure

```
├── SKILL.md          # The operational guide — your agent loads this on demand
├── reference.md      # Theoretical background and sources — for human readers; agent loads on demand for deep dives
└── README.md         # This file — for humans only, do NOT copy into your skill directory
```

> **Note:** When installing as a skill, copy only `SKILL.md` (required) and optionally `reference.md` into your agent's skill directory. `README.md` is for GitHub readers and should not be placed in the skill directory — per [Anthropic's skill authoring best practices](https://resources.anthropic.com/hubfs/The-Complete-Guide-to-Building-Skill-for-Claude.pdf).

### SKILL.md — The Operational Guide

This is what your AI agent reads when loaded. It contains:

- **Quick Checklist** — At-a-glance task checklist for every session
- **Mandatory Behaviors** — Non-negotiable rules: commit & push discipline, completion standard (no stubs, must run it), anti-laziness protocol, relentless forward progress until all TODOs are done
- **Phase 0: Orient** — Read agent instructions, check repo state, find progress artifacts
- **Phase 1: Reconnaissance** — Map architecture, find invariants, locate tests, calibrate effort to task size
- **Phase 1.5: Planning** — Expand prompts into product specs, separate planning from implementation for large tasks
- **Phase 2: Task Decomposition** — Break work into atomic, verifiable, reversible units; sprint contracts with concrete acceptance criteria; TODO discipline (append-only feature list, finish before switching)
- **Phase 3: Safe Execution** — Read-before-write, additive changes, immediate verification, adversarial self-review (Reviewer/QA/Architect role switch), commit & push after each feature, human escalation for high-risk decisions
- **Phase 4: Review and Harden** — Full regression check, doc updates, summary
- **Context Management** — Sub-agent-first investigation, just-in-time retrieval, file read deduplication, aggressive compaction, context anxiety detection, context reset vs compaction tradeoffs
- **Multi-Session Handoffs** — Progress files, feature lists, setup scripts, session start/end protocols
- **Separated Evaluation** — Anti-laziness verification patterns, evaluator calibration with few-shot examples, file-based inter-agent communication, single-agent evaluation loop (generate → evaluate → iterate)
- **Anti-Patterns** — Concrete "don't do this → do this instead" pairs
- **Failure Recovery** — Circuit breakers, 3-attempt limit, checkpoint/rewind, revert-and-rethink
- **Tool Usage** — Discover before using, phase-appropriate tool restriction, consolidate operations, handle denials as feedback

### reference.md — The Theory

Deep background for humans. Contains:

- **Curated sources** from Anthropic, OpenAI, LangChain, Thoughtworks, HumanLayer, Manus, OpenHands, OpenHarness, and the Claude Code source analysis
- **Distilled principles** synthesized from all sources
- **Multi-agent architecture patterns** (Two-Agent, Three-Agent, Orchestrator+Sub-Agents, Parallel Agent Teams, Autonomous Daemon)
- **Production implementation lessons** from the Claude Code codebase (layered compaction, prompt cache boundaries, asymmetric persistence, three-tier memory, file read deduplication)
- **Domain-specific application table** for web apps, APIs, CLIs, libraries, data pipelines, infrastructure, etc.

## Where the Knowledge Comes From

This project distills best practices from:

| Source | Key Contributions |
|--------|------------------|
| [Anthropic](https://www.anthropic.com/engineering) | Long-running agent harnesses, context engineering (context reset vs compaction), separated evaluation, multi-agent research system (orchestrator-worker, token scaling), tool design, sandboxing, auto mode, evals (eval awareness), parallel agent teams, Claude Agent SDK (agent loop: gather context → act → verify → repeat), 2026 Agentic Coding Trends Report |
| [OpenAI](https://openai.com/index/harness-engineering/) | Harness engineering, AGENTS.md, repo-local instructions, browser validation, agent-first architecture, 25-hour long-horizon Codex runs |
| [LangChain](https://blog.langchain.com/the-anatomy-of-an-agent-harness/) | Harness anatomy, framework vs runtime vs harness taxonomy, State of Agent Engineering 2026, agent observability via traces |
| [Thoughtworks / Martin Fowler](https://martinfowler.com/articles/exploring-gen-ai/harness-engineering.html) | Entropy management, reference anchoring, spec-driven development, quality loops, human-on-the-loop |
| [HumanLayer](https://www.humanlayer.dev/blog/12-factor-agents) | 12 Factor Agents, context backpressure, advanced context engineering, writing a good CLAUDE.md |
| [Manus](https://manus.im/blog/Context-Engineering-for-AI-Agents-Lessons-from-Building-Manus) | KV-cache locality, tool masking, keeping useful failures in context |
| [OpenHands](https://openhands.dev/blog) | Context condensation, prompt injection mitigation, trajectory critic verification, agent skill evaluation |
| [Inngest](https://www.inngest.com/blog/your-agent-needs-a-harness-not-a-framework) | State, retries, traces, and concurrency as first-class agent infrastructure |
| [OpenDev (arXiv)](https://arxiv.org/html/2603.05344v1) | Academic paper on CLI-native coding agents: compound architecture, multi-layer safety, context management |
| Claude Code source analysis ([ClaudeFast](https://claudefa.st), [Sabrina](https://www.sabrina.dev), [Paddo](https://paddo.dev), [Latent.Space](https://www.latent.space)) | 4-layer compaction, verification agent anti-laziness prompts, circuit breakers, 3-tier memory, KV-cache fork-join, quantified prompt constraints |
| [GitHub Spec Kit](https://github.com/github/spec-kit) | Spec-driven development toolkit for agents executing against explicit product and engineering specs |
| [AGENTS.md](https://github.com/agentsmd/agents.md) | Repo-local instruction format adopted by 20K+ repos |
| [OpenHarness (HKUDS)](https://github.com/HKUDS/OpenHarness) | First open-source harness framework; validates `.md` skill format as portable agent knowledge standard |
| [celesteanders/harness](https://github.com/celesteanders/harness) | Minimal generator+evaluator reference implementation with headless runner and JSON plan persistence |
| [Geoffrey Huntley](https://ghuntley.com/ralph/) | Ralph Wiggum technique: agent-in-a-while-loop, eventual consistency, tune via observation |
| [awesome-harness-engineering](https://github.com/walkinglabs/awesome-harness-engineering) | Curated index of the entire field |

## Principles at a Glance

1. **Reconnaissance before action** — understand the system before changing it
2. **Atomic, verifiable steps** — one concern per commit, verify immediately
3. **Additive over destructive** — extend, don't replace
4. **Immediate verification** — run it, don't just read it
5. **Context as working memory** — just-in-time, not just-in-case
6. **Structured handoffs** — progress files, feature lists, init scripts
7. **Separated evaluation** — don't grade your own homework
8. **Graceful failure with circuit breakers** — stop after 3 attempts, revert and rethink
9. **Progress over perfection** — working beats half-finished
10. **Tool design for agent ergonomics** — fewer tools, better designed
11. **Repo state as context** — git log, branch info, build status at session start
12. **Quantified constraints** — "≤25 words" beats "be concise"
13. **Adversarial self-verification** — "the implementer is an LLM — verify independently"
14. **Start monolithic** — single-agent loop until you hit a demonstrable ceiling
15. **Evolve harness with model** — strip scaffolding that new models no longer need
16. **Instructions as maps** — CLAUDE.md as TOC (~100 lines), not encyclopedia
17. **No placeholders** — full implementations, never stubs
18. **Mechanical invariants** — hooks, linters, and CI as deterministic guardrails, not just documentation
19. **Regenerate stale plans** — compare plan vs codebase, rebuild when drifted
20. **Interview before building** — clarify vague requirements into a written spec before planning
21. **Escalate, don't guess** — stop and ask the human on irreversible, security-sensitive, or ambiguous decisions
22. **Build to delete** — every harness component should be removable when models outgrow it

## Contributing

PRs welcome. If you've found a harness pattern that reliably improves agent performance, open an issue or PR with:

1. The pattern name and one-sentence description
2. Which source or experience it comes from
3. Before/after evidence (even anecdotal is fine)

## Acknowledgments

This project synthesizes the work of many teams, companies, and individuals across the AI engineering community. Equal thanks to all contributors — listed alphabetically:

[AGENTS.md](https://github.com/agentsmd/agents.md) · [Anthropic](https://www.anthropic.com/engineering) · [awesome-harness-engineering](https://github.com/walkinglabs/awesome-harness-engineering) · [celesteanders](https://github.com/celesteanders/harness) · [ClaudeFast](https://claudefa.st) · [Geoffrey Huntley](https://ghuntley.com/ralph/) · [GitHub Spec Kit](https://github.com/github/spec-kit) · [HumanLayer](https://www.humanlayer.dev) · [Inngest](https://www.inngest.com) · [LangChain](https://blog.langchain.com) · [Latent.Space](https://www.latent.space) · [Manus](https://manus.im) · [OpenAI](https://openai.com) · [OpenDev](https://arxiv.org/html/2603.05344v1) · [OpenHands](https://openhands.dev) · [OpenHarness / HKUDS](https://github.com/HKUDS/OpenHarness) · [Paddo](https://paddo.dev) · [Sabrina](https://www.sabrina.dev) · [Thoughtworks / Martin Fowler](https://martinfowler.com)

For detailed source attributions, see the ["Where the Knowledge Comes From"](#where-the-knowledge-comes-from) table above or [reference.md](reference.md).

## License

MIT
