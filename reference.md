# Harness Engineering — Reference

> **For humans and curious readers.** AI agents should use [SKILL.md](SKILL.md) — it contains everything needed for execution. This file provides the theoretical background, source material, and detailed rationale behind those practices.

## What Is Harness Engineering?

Harness engineering is the practice of shaping the environment, constraints, and workflow around an autonomous agent so it can complete complex tasks reliably. The term emerged from the AI engineering community (OpenAI, Anthropic, LangChain, Thoughtworks) in the context of making coding agents dependable for real-world software development.

The core insight: **when an agent fails at a task, the problem is usually the harness (workflow, context, constraints), not the model's intelligence.**

## Foundational Sources

### Primary Articles
- [OpenAI — Harness engineering: leveraging Codex in an agent-first world](https://openai.com/index/harness-engineering/) (Feb 2026) — Architectural constraints, repo-local instructions, browser validation, and telemetry for a large Codex-built application.
- [OpenAI — Run long horizon tasks with Codex](https://developers.openai.com/blog/run-long-horizon-tasks-with-codex/) (Feb 2026) — 25-hour uninterrupted Codex run, 13M tokens, 30K LOC generated. Practical lessons on AGENTS.md structure, checkpoint patterns, environment seeding, and "Extra High" reasoning for long autonomous sessions.
- [Anthropic — Effective harnesses for long-running agents](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents) (Nov 2025) — Initializer agents, feature lists, `init.sh`, self-verification, incremental progress, and handoff artifacts across context windows.
- [Anthropic — Harness design for long-running application development](https://www.anthropic.com/engineering/harness-design-long-running-apps) (Mar 2026) — GAN-inspired planner/generator/evaluator 3-agent architecture, sprint contracts, separated self-evaluation, and how context resets outperform compaction for long tasks.
- [Anthropic — Building a C compiler with a team of parallel Claudes](https://www.anthropic.com/engineering/building-c-compiler) (Feb 2026) — 16 Claude instances working in parallel on a shared codebase produced a 100K-line Rust C compiler. Key insights: task decomposition for parallel agents, shared-repo coordination, and 2000-session orchestration patterns.
- [Anthropic — How we built our multi-agent research system](https://www.anthropic.com/engineering/multi-agent-research-system) (Jun 2025) — Production multi-agent architecture: orchestrator-worker pattern with parallel subagents. Token usage explains 80% of performance variance. Six prompting principles for multi-agent coordination: teach delegation with explicit task boundaries, scale effort to query complexity, let agents self-improve tool descriptions (40% task-time reduction), start wide then narrow search strategy, guide thinking with extended thinking mode, and parallelize tool calls for up to 90% speed improvement. LLM-as-judge evaluation, stateful error compounding, and observability-first debugging in production.
- [Anthropic — Building effective agents](https://www.anthropic.com/engineering/building-effective-agents) (Dec 2024) — Workflows vs agents, composable patterns (prompt chaining, routing, parallelization, orchestrator-workers, evaluator-optimizer), and when structured systems outperform raw prompting.
- [LangChain — The Anatomy of an Agent Harness](https://blog.langchain.com/the-anatomy-of-an-agent-harness/) (Mar 2026) — Prompts + tools + middleware + orchestration + runtime = harness.
- [LangChain — Agent Frameworks, Runtimes, and Harnesses](https://blog.langchain.com/agent-frameworks-runtimes-and-harnesses-oh-my/) (Oct 2025) — Framework (LangChain) vs runtime (LangGraph) vs harness (DeepAgents): what belongs at each layer and why conflating them causes failures.
- [LangChain — On Agent Frameworks and Agent Observability](https://blog.langchain.com/on-agent-frameworks-and-agent-observability/) (Feb 2026) — Three generations of agent frameworks (chaining → orchestration → harness), why agent observability via traces is table stakes, and how building and testing converge in agent engineering.
- [LangChain — State of Agent Engineering 2026](https://www.langchain.com/state-of-agent-engineering) (Jan 2026) — Survey of 1,300+ professionals: 57% have agents in production, quality is the #1 barrier (32%), 89% have observability, multi-model usage is the norm.
- [Thoughtworks — Harness Engineering](https://martinfowler.com/articles/exploring-gen-ai/harness-engineering.html) (Feb 2026) — Context engineering + architectural constraints + entropy management ("garbage collection" against drift).
- [HumanLayer — Skill Issue: Harness Engineering for Coding Agents](https://www.humanlayer.dev/blog/skill-issue-harness-engineering-for-coding-agents) (Mar 2026) — Weak results from coding agents are harness problems, not model problems.
- [Inngest — Your Agent Needs a Harness, Not a Framework](https://www.inngest.com/blog/your-agent-needs-a-harness-not-a-framework) (Mar 2026) — State, retries, traces, and concurrency as first-class infrastructure.
- [OpenDev — Building AI Coding Agents for the Terminal](https://arxiv.org/html/2603.05344v1) (Mar 2026) — Academic paper on CLI-native coding agents: compound AI system architecture, multi-layer safety (prompt guardrails + runtime sandbox + code analysis), and context condensation strategies for long-horizon terminal tasks.

### Context Management
- [Anthropic — Effective context engineering](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents) (Sep 2025) — Treating the context window as a finite attention budget. Covers context rot, compaction, structured note-taking, sub-agent architectures, and "just-in-time" retrieval over pre-loading.
- [Anthropic — Context resets vs compaction](https://www.anthropic.com/engineering/harness-design-long-running-apps) (Mar 2026) — Context resets (clearing the window entirely + structured handoff) outperform compaction (in-place summarization) for long tasks. Compaction preserves continuity but doesn't eliminate "context anxiety" — models prematurely wrapping up work as they approach perceived context limits. A clean slate via reset removes this behavior at the cost of orchestration complexity. Newer models (Opus 4.5) reduced context anxiety enough to drop resets, but the pattern remains essential for models that exhibit it.
- [Manus — Context Engineering Lessons](https://manus.im/blog/Context-Engineering-for-AI-Agents-Lessons-from-Building-Manus) (Jul 2025) — KV-cache locality, tool masking, filesystem memory, keeping useful failures in-context.
- [Thoughtworks — Context Engineering for Coding Agents](https://martinfowler.com/articles/exploring-gen-ai/context-engineering-coding-agents.html) (Feb 2026) — Shaping the task environment so coding agents stay grounded and productive.
- [HumanLayer — Advanced Context Engineering](https://www.humanlayer.dev/blog/advanced-context-engineering) (Aug 2025) — Patterns for reducing context drift and making coding sessions easier to resume.
- [HumanLayer — Context-Efficient Backpressure](https://www.humanlayer.dev/blog/context-efficient-backpressure) (Dec 2025) — Preventing agents from burning context on low-value work.
- [OpenHands — Context Condensation](https://openhands.dev/blog/openhands-context-condensensation-for-more-efficient-ai-agents) (Apr 2025) — Bounded conversation memory that preserves goals, progress, critical files, and failing tests. Achieves 2x cost reduction with equivalent or better performance.
- [HumanLayer — Writing a Good CLAUDE.md](https://www.humanlayer.dev/blog/writing-a-good-claude-md) (Nov 2025) — Practical guide to repo-local instructions that agents can repeatedly follow.

### Constraints, Guardrails & Safe Autonomy
- [Anthropic — Claude Code auto mode: a safer way to skip permissions](https://www.anthropic.com/engineering/claude-code-auto-mode) (Mar 2026) — Learned policies that auto-approve 93% of safe permission prompts via transcript-monitoring classifier, eliminating approval fatigue while preserving safety. The middle ground between sandboxing and `--dangerously-skip-permissions`.
- [Anthropic — Claude Code sandboxing](https://www.anthropic.com/engineering/claude-code-sandboxing) (Oct 2025) — Reducing approval friction without losing control through sandboxing and policy design.
- [Anthropic — Code execution with MCP](https://www.anthropic.com/engineering/code-execution-with-mcp) (Nov 2025) — Giving agents controlled execution power through explicit, inspectable tool boundaries via MCP.
- [Anthropic — Writing effective tools for agents](https://www.anthropic.com/engineering/writing-tools-for-agents) (Sep 2025) — Tool interfaces that are easier for models to call correctly: namespacing, token-efficient responses, `response_format` enums, consolidating multi-step operations, and poka-yoke design.
- [OpenHands — Mitigating Prompt Injection](https://openhands.dev/blog/mitigating-prompt-injection-attacks-in-software-agents) (Jan 2026) — Confirmation mode, analyzers, sandboxing, and hard policies for reducing prompt-injection risk.
- [Thoughtworks — Assessing internal quality while coding with an agent](https://martinfowler.com/articles/exploring-gen-ai/ccmenu-quality.html) (Jan 2026) — Moving quality checks into the loop instead of after-the-fact manual review.
- [Thoughtworks — Anchoring to a reference application](https://martinfowler.com/articles/exploring-gen-ai/anchoring-to-reference.html) (Feb 2026) — Constraining agents with concrete exemplars for consistent output via MCP-served code samples and drift detection.
- [Thoughtworks — Humans and Agents in Software Engineering Loops](https://martinfowler.com/articles/exploring-gen-ai/humans-and-agents.html) (Mar 2026) — Where humans should be "on the loop" (managing the working loop) rather than "in the loop" (micromanaging artifacts) or "out of the loop" (vibe coding).

### Evaluation & Observability
- [Anthropic — Demystifying evals for AI agents](https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents) (Jan 2026) — Task, trial, grader, transcript, outcome definitions. Three grader types (code-based, model-based, human). Capability evals vs regression evals.
- [Anthropic — Quantifying infrastructure noise in agentic coding evals](https://www.anthropic.com/engineering/infrastructure-noise) (Feb 2026) — Runtime configuration alone can move benchmark scores by 6+ percentage points. Agentic evals measure the harness+model together, not model alone.
- [Anthropic — Eval awareness in Claude Opus 4.6's BrowseComp performance](https://www.anthropic.com/engineering/eval-awareness-browsecomp) (Mar 2026) — Agents can independently hypothesize they are being evaluated, identify the benchmark, and search for leaked answers. A novel contamination pattern where the model exhibits meta-awareness of the evaluation context. Implications for eval design: treat agent resourcefulness as a first-class threat model.
- [Anthropic — Designing AI-resistant technical evaluations](https://www.anthropic.com/engineering/AI-resistant-technical-evaluations) (Jan 2026) — Each new model generation solves more of a previously-hard evaluation. Core insight: constrain evaluations via **time, tools, or environment** rather than just problem difficulty. Designed a take-home test that survived multiple model generations by requiring real hardware interaction (simulated accelerator) that models couldn't shortcut. Takeaway for harness engineering: as models improve, **continuously re-evaluate your verification criteria** — what was hard to fake yesterday may be trivially gamed today.
- [OpenAI — Testing Agent Skills Systematically with Evals](https://developers.openai.com/blog/eval-skills/) — Turning agent traces into repeatable evals with JSONL logs and deterministic checks.
- [OpenHands — How to Evaluate Agent Skills](https://openhands.dev/blog/evaluating-agent-skills) (Mar 2026) — Bounded tasks, deterministic verifiers, no-skill baselines, and trace review. Inspired by SkillsBench.
- [OpenHands — Learning to Verify AI-Generated Code](https://openhands.dev/blog/20260305-learning-to-verify-ai-generated-code) (Mar 2026) — Layered verification stack using trajectory critics trained on production traces for reranking, early stopping, and review-time quality control.
- [LangChain — Improving Deep Agents with harness engineering](https://blog.langchain.com/improving-deep-agents-with-harness-engineering/) (Feb 2026) — Harness changes alone moved from Top 30 to Top 5 on Terminal Bench 2.0. Self-verification and tracing were the biggest levers.
- [LangChain — Evaluating Deep Agents](https://blog.langchain.com/evaluating-deep-agents-our-learnings/) (Dec 2025) — Single-step, full-run, and multi-turn eval design for stateful agents across four shipped applications.
- [LangChain — From Traces to Insights: Understanding Agent Behavior at Scale](https://blog.langchain.com/from-traces-to-insights-understanding-agent-behavior-at-scale/) (Jan 2026) — Agent traces as the foundational artifact for debugging, monitoring, and evals. The trace is to agents what code is to traditional software.

### Production Harness Anatomy (Claude Code Source Analysis)

Multiple independent analyses of the leaked Claude Code source (512K LOC, 1,900 files of TypeScript) revealed how a production harness works at scale. Key architectural insight: "The model is the most interchangeable part. The harness is where years of production experience live."

- [Claude Code Source Leak: Everything Found — ClaudeFast](https://claudefa.st/blog/guide/mechanics/claude-code-source-leak) (Mar 2026) — Comprehensive catalog: 19 permission-gated tools, 44 feature flags, 5 compaction strategies, 14 prompt cache-break vectors, 23 bash security validators, 3 subagent execution models (Fork/Teammate/Worktree), KAIROS autonomous daemon, autoDream memory consolidation (4-phase: orient → gather → consolidate → prune), ULTRAPLAN remote planning (30-min Opus sessions).
- [Comprehensive Analysis of Claude Code Source Leak — Sabrina](https://www.sabrina.dev/p/claude-code-source-leak-analysis) (Mar 2026) — Verification agent anti-laziness prompts ("the implementer is an LLM — verify independently"), circuit breaker pattern (MAX_CONSECUTIVE_FAILURES=3 after 250K wasted API calls/day), A/B-tested prompt quantification ("≤25 words" beats "be concise" by 1.2% token reduction), compaction security gap.
- [The Claude Code Leak: What the Harness Actually Looks Like — Paddo](https://paddo.dev/blog/claude-code-leak-harness-exposed/) (Mar 2026) — Three-layer memory architecture (MEMORY.md index → topic files → searchable transcripts), KV-cache fork-join for free sub-agent parallelism, context entropy management as the hard problem separating usable agents from demos.
- [AINews: The Claude Code Source Leak — Latent.Space](https://www.latent.space/p/ainews-the-claude-code-source-leak) (Mar 2026) — Community synthesis: repo state in context, aggressive cache reuse, file read deduplication, structured session memory, subagents. The real moat is harness engineering.

### Workflow Design & Agent Files
- [AGENTS.md format](https://github.com/agentsmd/agents.md) — Repo-local instructions that tell agents how to work inside a codebase. Adopted by 20K+ repos.
- [GitHub Spec Kit](https://github.com/github/spec-kit) (Sep 2025) — Spec-driven development toolkit for agents executing against explicit product and engineering specs.
- [Thoughtworks — Understanding SDD: Kiro, spec-kit, and Tessl](https://martinfowler.com/articles/exploring-gen-ai/sdd-3-tools.html) (Mar 2026) — Why strong specs make AI-assisted software delivery more dependable.
- [Claude Code best practices](https://code.claude.com/docs/en/best-practices) (Apr 2025, continuously updated) — Operational best practices for agentic coding: CLAUDE.md authoring (concise, check into git, prune regularly), skills as on-demand knowledge, hooks for deterministic guarantees, custom subagents with scoped tools, plugins as distributable bundles, context management (`/clear` between tasks, `/compact` with focus instructions, `/btw` for side questions), rewind/checkpoints for reversible exploration, and verification-first development (run tests, compare screenshots, validate outputs).
- [OpenAI Codex best practices](https://developers.openai.com/codex/learn/best-practices/) (Mar 2026) — AGENTS.md structure, repo layout guidance, build/test/lint commands, and engineering conventions for Codex.
- [HumanLayer — 12 Factor Agents](https://www.humanlayer.dev/blog/12-factor-agents) (Apr 2025) — Explicit prompts, own your context window, state ownership, clean pause-resume, small focused agents, stateless reducer pattern.
- [Anthropic — 2026 Agentic Coding Trends Report](https://resources.anthropic.com/2026-agentic-coding-trends-report) (Jan 2026) — Eight trends reshaping software development: shifting engineering roles to agent orchestration, multi-agent coordination, human-AI review loops, and quality evaluation as the new bottleneck.
- [Geoffrey Huntley — The Ralph Wiggum Technique](https://ghuntley.com/ralph/) (Jul 2025) — Agent-in-a-while-loop pattern: `while :; do cat PROMPT.md | claude-code ; done`. One task per loop, deterministic context loading, trust the agent to prioritize, tune via observation. Key insight: "eventual consistency" — most issues resolve through more loops with better-tuned prompts.

### Agent Extensibility (Skills, Hooks, Plugins)

Modern agent platforms expose multiple extensibility layers that serve as concrete implementation surfaces for harness engineering patterns:

- **Skills** (on-demand knowledge): Markdown files loaded into context only when relevant. The agent reads them at task start and follows the instructions. This is the delivery mechanism this project uses — `.md` files in a skills directory. Claude Code, Cursor, CodeBuddy, OpenClaw, and OpenHarness all support this pattern, making it the emerging portable standard for agent knowledge.
- **Hooks** (event-driven automation): JSON-configured rules that fire automatically at specific workflow events (e.g., after every file edit, before every commit). Unlike instruction files which are advisory, hooks are deterministic — they guarantee the action happens. Examples: run eslint after every edit, block writes to `dist/`, auto-format on save. Hooks are the primary mechanism for enforcing invariants mechanically (see Principle 18).
- **Custom subagents**: Scoped agents with restricted tool access for specific tasks (e.g., a "reviewer" subagent that can only read files and run tests, not edit). Enables separated evaluation without giving the evaluator write access.
- **Plugins** (distributable bundles): Packages that combine skills + hooks + commands + MCP server configs into a single installable unit. The distribution mechanism for sharing harness components across teams and projects.
- [Anthropic — Enabling Claude Code to work more autonomously](https://www.anthropic.com/news/enabling-claude-code-to-work-more-autonomously) (Sep 2025) — SDK support for subagents and hooks, making agent workflows customizable for specific domains.

### Open-Source Harness Implementations
- [HKUDS/OpenHarness](https://github.com/HKUDS/OpenHarness) (Apr 2026) — First open-source Python harness framework implementing the full agent loop pattern: 43 tools, on-demand skill loading (`.md` files compatible with `anthropics/skills`), CLAUDE.md discovery, context auto-compaction, MEMORY.md persistence, multi-level permission governance, subagent spawning, and team coordination. Supports CLI integration with OpenClaw, Cursor, and nanobot. Validates that the skill-file-as-knowledge pattern (what this project uses) is the emerging standard for portable agent instructions.
- [celesteanders/harness](https://github.com/celesteanders/harness) (Mar 2026) — Minimal generator+evaluator reference implementation for Claude Code with headless runner and JSON plan persistence.

## Distilled Principles

### 1. Reconnaissance Before Action
Understand the system before changing it. Map architecture, identify invariants, locate tests. This is the highest-ROI activity — an agent that understands the system makes fewer mistakes.

### 2. Atomic, Verifiable Steps
Every change should be small enough to verify independently. If you can't tell whether a change worked, it's too big. Break it down. Anthropic found that working **one feature at a time** was critical to preventing agents from trying to do too much at once.

### 3. Additive Over Destructive
Extend existing systems rather than replacing them. New code paths alongside old ones. Feature flags. Backward compatibility. The existing system earned its stability — respect that.

### 4. Immediate Verification (Self-Testing)
Don't queue up verification for later. After each change: lint, test, trace. Fix issues before moving on. Accumulated unverified changes are the primary cause of agent derailment. Agents tend to **mark features as complete without proper end-to-end testing** — explicit prompting to run the code and test as a user dramatically improves results.

### 5. Context as Working Memory
The context window is finite and suffers from **context rot** — as tokens increase, recall accuracy decreases. Use it intentionally:
- Re-read files rather than relying on stale memory.
- Keep TODO lists and progress files current as your source of truth.
- Summarize completed work to free up mental space for upcoming work.
- Don't load irrelevant context "just in case."
- Use sub-agents for investigation-heavy tasks to keep the main context clean.
- Prefer "just-in-time" retrieval (reading files on demand) over pre-loading everything.
- Keep useful failures in context — they prevent repeated mistakes (Manus).

### 6. Structured Handoffs Across Sessions
Long-running tasks span multiple context windows. Each new session starts with no memory. Bridge the gap with:
- **Progress files** (e.g., `progress.md`) — What's been done, what's next, key decisions.
- **Feature lists** (e.g., `feature_list.json`) — Structured JSON with pass/fail status for each feature. Harder for agents to accidentally modify than Markdown.
- **Git commits** — Descriptive messages that serve as a log of incremental progress.
- **init.sh** — Setup script that lets the next session quickly get the environment running.
- Each session starts by reading these artifacts and running a basic smoke test before making new changes.

### 7. Separated Evaluation
Agents that evaluate their own work tend to be overly generous. Separating generation from evaluation is a powerful lever:
- A **generator** agent builds the implementation.
- An **evaluator** agent tests the output against concrete criteria using real tools (browser automation, API calls, etc.).
- The evaluator's critique feeds back to the generator for iteration.
- Sprint contracts (agreeing on "done" criteria before coding) keep both aligned.

### 8. Graceful Failure with Circuit Breakers
When something breaks: stop, diagnose, fix or revert. Never push forward through errors hoping they'll resolve. Every unaddressed error makes the next step less reliable. Any automated retry loop MUST have a maximum consecutive failure limit (default: 3). Claude Code discovered 250K wasted API calls/day from unbounded compaction retries before adding this.

### 9. Progress Over Perfection
Complete the core task before polishing. A working implementation with rough edges is more valuable than a half-finished perfect design. File follow-ups for improvements that aren't blocking.

### 10. Tool Design for Agent Ergonomics
Tools define the contract between deterministic systems and non-deterministic agents. Design for the agent's affordances:
- Consolidate multi-step operations into single tools.
- Return token-efficient, high-signal responses. Use `response_format` enums (concise vs detailed).
- Namespace tools clearly to avoid ambiguity when many tools are available.
- Use human-readable identifiers over cryptic UUIDs.
- Fewer, well-designed tools outperform many overlapping ones.
- Feed permission denials back as tool results so the model can adapt its approach.

### 11. Repo State as Context
Put the working environment into context at session start: recent git commits, current branch, changed files, build status. This grounds the agent in reality rather than assumptions. Claude Code injects git status, branch info, and recent commit history into every session. OpenAI's Codex embeds AGENTS.md as repo-local instruction.

### 12. Quantified Constraints Over Qualitative Instructions
"Be concise" is vague. "Keep text between tool calls to ≤25 words" is actionable. Anthropic A/B tested this and found ~1.2% token reduction with hard numeric constraints. Apply everywhere: verification summaries ("endpoint returns 200" not "looks good"), commit messages, progress updates.

### 13. Adversarial Self-Verification
Self-review is unreliable. Claude Code's verification agent has explicit anti-laziness prompts: "reading is not verification — run it", "the implementer is an LLM — verify independently", "probably is not verified — run it." The verification agent must use real tools against the running system, never just inspect code.

### 14. Start Monolithic, Add Agents When Needed
Start with a single-agent loop before reaching for multi-agent orchestration. Multi-agent systems introduce microservice-like complexity compounded by non-determinism. Only add agents when you hit a specific, demonstrable ceiling. Three similar lines of code are better than a premature abstraction; a single-agent loop is better than a multi-agent system until the single agent provably fails.

### 15. Evolve the Harness with the Model
Every harness component encodes an assumption about what the model can't do. These assumptions go stale as models improve. When a new model lands: strip scaffolding that's no longer load-bearing, add new components for newly possible capabilities, and test by removing one component at a time. The evaluator adds most value at the edge of model capability — for easy tasks it becomes overhead.

### 16. Instruction Files as Maps, Not Encyclopedias
Keep `CLAUDE.md` / `AGENTS.md` short (~100 lines). Use it as a table of contents pointing to deeper sources in a structured `docs/` directory. A giant instruction file crowds out the task prompt and rots quickly. Structure for progressive disclosure: agents start with a stable entry point and are taught where to look.

### 17. Prevent Placeholder Implementations
Agents are biased toward stub/minimal implementations because compiling code triggers their reward function. Explicitly instruct against placeholders and enforce full implementations. Use strong language if needed ("implement fully, no TODOs, no stubs").

### 18. Enforce Invariants Mechanically
Don't rely on documentation alone to maintain code quality. Use linters, structural tests, CI checks, and **hooks** (agent workflow hooks, git hooks) with custom error messages that include remediation instructions so the agent can self-fix. Unlike instruction files which are advisory, hooks are deterministic and guarantee the action happens — they run automatically at specific points in the workflow (e.g., run eslint after every file edit, block writes to protected directories). Encoded rules become multipliers in agent-generated codebases.

### 19. Regenerate Plans Periodically
Plans and TODO lists drift as implementation progresses. Periodically delete and regenerate them by having the agent compare the current codebase against the specification. This prevents following stale or contradictory plans. Also allow agents to self-improve their instruction files with learnings about build, test, and run commands.

### 20. Interview Before Building on Vague Requirements
When the task is underspecified or ambiguous, interview the user before planning. Ask about technical constraints, edge cases, tradeoffs, and what "done" looks like — including the hard parts they might not have considered (implementation details, UI/UX, error handling). Write the answers into a spec file (e.g., `SPEC.md`), then start a fresh context focused on implementation. A clean session with a written spec outperforms a long session with accumulated clarifications.

## Applying to Different Domains

| Domain | Reconnaissance Focus | Key Invariants | Verification Method |
|--------|---------------------|----------------|-------------------|
| Web app (React, Vue) | Component tree, state management, routing | UI renders, tests pass, no console errors | Dev server + browser automation |
| Backend API | Route handlers, DB schema, auth middleware | All endpoints respond correctly, migrations work | Test suite + curl/httpie |
| CLI tool | Argument parsing, subcommands, output format | Exit codes, stdout format, backward compatibility | Shell invocation |
| Library/SDK | Public API surface, type signatures | Existing consumers don't break | Type checker + test suite |
| Legacy C/C++ | Build system, memory management, global state | No crashes, no memory leaks, ABI compatibility | Compile + valgrind + smoke test |
| Data pipeline | DAG structure, schema evolution, idempotency | Data integrity, no duplicates, backward-compatible schemas | Integration test + data validation |
| Infrastructure | Terraform/K8s manifests, secrets, networking | Services stay reachable, rollback works | Plan/diff + staged rollout |
| Long-running app | Feature list, progress state, session artifacts | Incremental progress, no regressions, clean handoff | Evaluator agent + browser QA |

## Multi-Agent Architecture Patterns

Based on Anthropic's research, the Claude Code source, and the broader community:

### Initializer + Coding Agent (Two-Agent)
- **Initializer**: First session. Expands the prompt into a feature list, writes `init.sh`, creates initial git commit, sets up progress tracking.
- **Coding Agent**: Every subsequent session. Reads progress files + git log, runs smoke test, picks one feature, implements it, commits, updates progress.
- Best for: Structured coding tasks with clear feature decomposition.

### Planner + Generator + Evaluator (Three-Agent)
- **Planner**: Takes a brief prompt and expands into a full product spec with ambitious scope. Focuses on product context, not granular technical implementation.
- **Generator**: Works in sprints, one feature at a time. Implements against sprint contracts. Self-evaluates before handing off to QA.
- **Evaluator**: Uses real tools (Playwright, curl, etc.) to test the running application. Grades against criteria with hard thresholds. Failed sprints get detailed feedback for reiteration.
- Before each sprint, generator and evaluator negotiate a **sprint contract**: what "done" looks like, tested behaviors.
- Best for: Complex applications requiring subjective quality judgment and long autonomous runs.

### Orchestrator + Sub-Agents
- A coordinating agent maintains the high-level plan while sub-agents perform focused tasks in separate context windows.
- Each sub-agent explores extensively but reports back concise summaries.
- The orchestrator's context stays clean; the sub-agents handle the heavy lifting.
- Three sub-agent delegation models (from Claude Code source):
  - **Fork**: Full context clone via KV-cache fork. Sub-agent has complete parent context. Best for exploration tasks.
  - **Teammate**: Shared workspace but different focus areas. Best for parallel implementation of independent features.
  - **Worktree**: Git worktree isolation. Sub-agent works on a separate branch. Best for risky changes that might need to be discarded.
- Best for: Tasks requiring deep exploration alongside focused implementation.

### Parallel Agent Teams (New in 2026)
- Multiple agent instances work **simultaneously on a shared codebase** without active human intervention. Anthropic demonstrated this with 16 Claude instances building a 100K-line C compiler across ~2,000 sessions.
- Key coordination patterns: task decomposition into independent modules, shared git repo as coordination substrate, human supervisor providing periodic direction, and clear module ownership to avoid merge conflicts.
- Different from Orchestrator+Sub-Agents: no central orchestrating agent — agents work peer-to-peer with human oversight.
- Best for: Large-scale greenfield projects where modules are naturally parallelizable.

### Autonomous Daemon (KAIROS Pattern)
- An always-on background agent that receives periodic **tick prompts** and independently decides whether to act.
- Key design constraints: append-only logging, blocking budget per decision cycle, triple-gated consolidation (time + session count + file lock), stale lock guard.
- Best for: Continuous monitoring (PR reviews, CI failures, security alerts) and background maintenance tasks.

## Production Implementation Lessons (from Claude Code Source)

These are infrastructure-level patterns observed in Claude Code's 512K LOC codebase. They inform harness design at the platform level:

- **Layered context compaction**: A single compaction strategy is fragile. Claude Code uses four layers: proactive (pre-emptive summarization), reactive (catch `prompt_too_long`, compact, retry), snip (truncation for headless/SDK mode), and context collapse (compress verbose tool results mid-conversation with reversible commits).
- **Prompt cache via static/dynamic split**: Split the system prompt at a deterministic `DYNAMIC_BOUNDARY` marker. Static content (instructions, tool definitions) cached globally; dynamic content (user config, repo state) injected per-session. Hash the static prefix for maximum cache hits.
- **Asymmetric session persistence**: Persist user messages synchronously (ensures resumability if process dies), assistant messages asynchronously (already durable in API response). Production scar from lost sessions.
- **Three-tier memory**: MEMORY.md as lightweight index → topic files loaded on demand → full session transcripts searchable but never loaded into context. Add periodic consolidation (autoDream): merge, deduplicate, prune, remove contradictions.
- **File read deduplication**: Track which files are current in context. Don't re-read unchanged files within the same turn. Claude Code uses an LRU cache (100 files, 25MB) with snapshot support for undo.
- **Speculation and prefetch**: Pre-compute likely next responses while the user types. Prefetch relevant memory files during model streaming so they're ready when tools execute.
