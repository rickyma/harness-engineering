---
name: harness-engineering
description: >-
  Disciplined methodology for AI agents executing large, multi-step software
  tasks reliably. Use when working on complex refactors, feature development,
  optimization, migration, or any task spanning many files or requiring
  sustained focus. Ensures safety, traceability, and completeness across
  long-running sessions, including multi-context-window handoffs.
---

# Harness Engineering

A systematic approach for completing large software tasks without breaking things — across one or many sessions.

**At the start of every task, re-read Phase 0 below.** This file is self-contained — everything you need is here.

## Quick Checklist (every task)

- [ ] Read agent instruction file (CLAUDE.md / .cursor/rules / AGENTS.md)
- [ ] Check `git status` and `git log --oneline -10`
- [ ] Read progress.md / feature_list.json if they exist
- [ ] Calibrate effort — match harness depth to task size (see Phase 1)
- [ ] **Explore and plan before coding** — understand the system first, then create a plan; don't jump to implementation
- [ ] Map architecture before coding
- [ ] Break into atomic tasks; create TODO list
- [ ] For each task: read → implement → verify → commit → update progress
- [ ] After all tasks: re-read modified files, run full tests, update docs

## When to Apply

- Task touches 3+ files or requires 5+ distinct steps
- Codebase is unfamiliar or partially understood
- Stability matters (production system, long-running service, shared codebase)
- Task spans a long session where context can drift
- Task is too large for a single context window (multi-session work)

**Skip this methodology for**: Single-file bug fixes, documentation-only changes, simple config tweaks, or any task completable in < 3 steps.

## Phase 0: Orient

Before anything else, ground yourself in the current state of the repo. **Explore first, then plan, then code** — resist the urge to jump straight to implementation.

1. **Read the agent instruction file** — Look for `CLAUDE.md`, `.cursor/rules/`, `AGENTS.md`, `CONVENTIONS.md`, or equivalent. These contain repo-specific instructions: build commands, test commands, coding conventions, architectural decisions. Follow them.
2. **Check repo state** — Run `git status`, `git log --oneline -10`, check the current branch. Know what's changed recently and what state the working tree is in.
3. **Read progress artifacts** — If `progress.md`, `feature_list.json`, or similar exist, read them. You may be continuing work from a previous session.
4. **Clarify ambiguous requirements** — If the task is vague or underspecified, interview the user before planning. Ask about technical constraints, edge cases, tradeoffs, and what "done" looks like. Write the answers into a spec file (e.g., `SPEC.md`) and start a fresh context focused on implementation.

If any of these files exist, their instructions take precedence over generic guidance below.

## Phase 1: Reconnaissance

Before writing any code, build a mental model of the system.

1. **Map the architecture** — Identify entry points, core loops, data flows, module boundaries. Read key files, not all files.
2. **Identify invariants** — What MUST remain true? (API contracts, data formats, test suites, config compatibility)
3. **Find the blast radius** — For each planned change, list which components it affects.
4. **Catalog existing patterns** — How does the codebase handle errors, logging, configuration, testing? Follow those patterns.
5. **Locate the test/verification surface** — Unit tests, integration tests, linter configs, CI scripts, manual smoke-test procedures.

Use **sub-agents** for investigation-heavy reconnaissance if your agent runtime supports them. They explore in separate context windows and report back summaries, keeping your main context clean. Delegation models vary by platform:
- **Fork**: Full context inheritance for exploration tasks needing complete situational awareness.
- **Teammate**: Shared workspace, different focus — for parallel investigation of independent areas.
- **Worktree**: Git-isolated branch for risky exploratory changes that might be discarded.

If sub-agents aren't available, use the main context but be disciplined about collapsing investigation output to conclusions before proceeding with implementation.

### Calibrate Effort

Not every task needs the full harness. Match investment to task size:

- **Small tasks** (1-2 files, clear change): Skip sub-agents. Brief reconnaissance, implement directly. Verify and commit.
- **Medium tasks** (3-10 files, some ambiguity): Full Phase 1 reconnaissance. Use sub-agents for exploration if available. Create a TODO list.
- **Large tasks** (10+ files, unfamiliar codebase, multi-session): Full harness with progress files, feature lists, and setup scripts. Use sub-agents aggressively. Plan for multi-session handoffs.

Don't over-invest in harness infrastructure for tasks that don't need it.

Output: a clear TODO list with ordered, scoped tasks before any implementation begins.

## Phase 1.5: Planning (Large Tasks)

For large or greenfield tasks, separate **planning** from **implementation**:

1. **Expand the prompt into a product spec** — Take the user's brief description (even 1-4 sentences) and expand it into a full spec with features, user flows, and scope. Be ambitious but stay at the product level — don't specify granular implementation details upfront.
2. **Constrain deliverables, not paths** — **Key lesson: errors in over-specified plans cascade into implementation.** Define *what* to build (features, acceptance criteria), not *how* to build it. Let the implementation agent figure out the technical path. This avoids brittle plans that break on contact with the actual codebase.
3. **Separate the planner from the builder** — If your runtime supports multiple agents, use a dedicated planner agent that writes the spec, then hand it off to a generator agent that implements one feature at a time. The planner focuses on product context and high-level technical design; the generator focuses on code.

For medium tasks, you can combine planning and implementation in the same agent — but still write down the plan (as a TODO list or spec) before coding. For small tasks, skip this phase entirely.

## Phase 2: Task Decomposition

Break the work into **atomic units** — each one is independently:
- Implementable in a single focused step
- Verifiable (compiles, passes tests, or can be manually confirmed)
- Reversible (can be undone without cascading effects)

Rules:
- One logical concern per change. Don't mix a refactor with a feature.
- If a task requires changing 3+ interconnected systems simultaneously, decompose further.
- Order tasks so each builds on verified previous work (no forward dependencies on unverified code).

For large tasks that will span multiple sessions, create **structured artifacts**:
- **Feature list** (JSON preferred over Markdown — harder to accidentally corrupt): enumerate all features/requirements with pass/fail status.
- **Progress file** (`progress.md`): running log of what's been done, decisions made, and what's next.
- **Setup script** (`init.sh` / `setup.ps1`): reproducible environment bootstrap so any new session can get running quickly.
- Commit the initial scaffolding to git as the first commit.

### Sprint Contracts

For any non-trivial feature, define a **sprint contract** before implementing — this is a critical lever for quality:
1. What "done" looks like in concrete, testable terms.
2. Specific behaviors to verify (e.g., "POST /users with missing name returns 400").
3. Hard pass/fail thresholds for each criterion.

The contract bridges the gap between high-level specs and testable implementation. When using separated evaluation (see *Separated Evaluation* below), the generator proposes what it will build and how success will be verified; the evaluator reviews the proposal before any code is written. Even without a separate evaluator, writing the contract forces you to think about success criteria before coding — preventing scope drift and "vibes-based" completion.

Example sprint contract for a "User Authentication" sprint:
```
Sprint 3: User Authentication
Deliverables:
  - Login page with email/password form
  - POST /auth/login endpoint returning JWT
  - Protected route middleware

Acceptance criteria (hard pass/fail):
  1. POST /auth/login with valid credentials → 200 + JWT token (3 segments)
  2. POST /auth/login with wrong password → 401 + {"error": "Invalid credentials"}
  3. POST /auth/login with missing email → 400 + validation error
  4. GET /dashboard without token → 302 redirect to /login
  5. GET /dashboard with valid token → 200 + dashboard content
  6. Login form submits and redirects to /dashboard on success
  7. Login form shows error message on failure

Threshold: all 7 criteria must pass. Any failure → sprint fails with specific feedback.
```

When using a separate evaluator, provide 2-3 scored example evaluations in its prompt to anchor its judgment to your quality standards and reduce score drift across iterations.

## Phase 3: Safe Execution

For each atomic task:

### 3a. Read Before Write
Always read the target code before modifying it. Never assume structure from memory. Re-read files you haven't looked at recently — stale memory is the #1 source of incorrect edits.

### 3b. Preserve Original Behavior
- Prefer **additive** changes (new functions, new branches) over rewriting existing logic.
- When modifying existing behavior, keep the original path reachable via a toggle, flag, or parameter.
- Guard new code with bounds checks, null guards, and safe defaults.

Example — adding retry logic:
```
# BAD: rewrite the existing function
def fetch_data(url):
    for attempt in range(3):          # original callers now get retry behavior they didn't ask for
        ...

# GOOD: add a new function, keep the original intact
def fetch_data(url):                  # unchanged — existing callers unaffected
    return requests.get(url)

def fetch_data_with_retry(url, retries=3):   # new code path, opt-in
    for attempt in range(retries):
        ...
```

### 3c. Minimize Blast Radius
- Don't change public interfaces unless that IS the task.
- Don't change data serialization formats without migration.
- Don't introduce new dependencies unless explicitly required.
- Don't change build configuration as a side effect.

### 3d. Verify Immediately (Self-Test)
After each change:
1. Check for linter/compiler errors in modified files.
2. Run relevant tests if available.
3. If no tests exist, mentally trace the change through all callers/consumers.
4. For UI changes: use browser automation (e.g., Playwright MCP) or screenshots to verify end-to-end behavior as a user would. **Do not mark a feature as complete based only on code review — test it running.**
5. Fix any introduced issues before moving to the next task.
6. Be specific in verification summaries:

```
# BAD verification summary
"Looks good, the code is correct."

# GOOD verification summary
"GET /api/users returns 200 with 3 users. POST /api/users with valid body returns 201. POST with missing 'name' returns 400 with error message."
```

### 3e. Commit Incrementally
After each verified change:
1. Commit to git with a descriptive message explaining what changed and why.
2. This creates recovery points — if something goes wrong later, you can revert to a known-good state.

### 3f. Track Progress
Update the TODO list and progress file after each task completes. Mark done, note any discovered follow-ups. This is critical for multi-session work — the next session will use these artifacts to get up to speed.

## Phase 4: Review and Harden

After all tasks are implemented:

1. **Re-read all modified files** — Look for inconsistencies, missed edge cases, style violations.
2. **Check for regressions** — Verify invariants identified in Phase 1 still hold.
3. **Run full verification** — Execute the complete test suite, not just targeted tests.
4. **Update docs/tests** — If the project has README, CHANGELOG, tests, or inline docs that reference changed behavior, update them.
5. **Enforce invariants mechanically** — Don't rely on documentation or memory alone to maintain quality. Use hooks, linters, and CI checks as deterministic guardrails:
   - **Hooks** (e.g., Claude Code hooks, git hooks) run automatically at specific points in the workflow — unlike instruction files which are advisory, hooks are deterministic and guarantee the action happens. Example: run eslint after every file edit, block writes to protected directories.
   - **Structural tests** with custom error messages that include remediation instructions so the agent can self-fix.
   - Encoded rules become multipliers in agent-generated codebases.
6. **Summarize changes** — Provide a clear, structured summary of what changed and why.

## Context Management (Long Sessions)

Your main context window is your most precious resource. Guard it.

For tasks that take many turns:

- **Prefer sub-agents for all investigation** — When you need to explore the codebase, read many files, research a question, or review code, **always delegate to a sub-agent** if your runtime supports it. Sub-agents run in separate context windows and report back concise summaries. Your main context stays focused on planning and implementation. This is one of the most powerful tools available — use it aggressively.
- **Anchor on the TODO list** — It is your single source of truth for what's done and what remains.
- **Re-read before resuming** — If you haven't looked at a file recently, read it again rather than relying on memory.
- **Checkpoint often** — After completing a coherent group of changes, summarize the current state before continuing.
- **Don't drift** — If you discover a tangential issue, note it as a follow-up rather than fixing it inline (unless it blocks current work).
- **Prefer just-in-time retrieval** — Don't pre-load files "just in case." Read them when you need them. Keep lightweight references (file paths, function names) and load content on demand.
- **Deduplicate file reads** — If you've already read a file this turn and it hasn't changed, don't read it again. Track what's current in context.
- **Compact aggressively** — When verbose tool output (long file contents, build logs, test output) is no longer needed for current work, mentally "collapse" it — retain the conclusion, discard the details.
- **Keep useful failures in context** — A failed approach is valuable information that prevents repeating the same mistake. Don't discard error messages and failed hypotheses too eagerly.

### Context Reset vs Compaction

Two strategies for managing a filling context window — choose based on situation:

- **Compaction** (summarizing earlier conversation in-place): Preserves continuity but doesn't give the agent a clean slate.
- **Context reset** (clearing the window entirely, starting a fresh agent with structured handoff artifacts): Eliminates context anxiety and gives a clean slate. The cost is orchestration complexity — the handoff artifact must carry enough state for the next agent to pick up cleanly.

**Context anxiety** is a failure mode where agents prematurely wrap up or rush work as they approach perceived context limits — cutting corners, skipping verification, or declaring tasks "done" early. Compaction does not fix this because the agent still perceives itself as deep into a long conversation. Only a full context reset (clean slate) eliminates it. Watch for these symptoms:
- Verification steps becoming shorter or skipped entirely as the session progresses
- The agent suggesting "we can finish the rest later" when there's no actual reason to stop
- Declining quality or increasing shortcuts in later tasks compared to earlier ones

**When to use which:**
- Short-to-medium tasks where context fits comfortably → compaction is sufficient.
- Long tasks (multi-hour, many features) or when quality noticeably degrades mid-session → prefer context reset with structured handoff (progress file + feature list + git commits).
- **Model-dependent**: This behavior varies significantly by model. Some models (e.g., Sonnet 4.5) exhibit strong context anxiety and need resets. Others (e.g., Opus 4.5+) handle long contexts well enough that compaction alone works — Anthropic was able to drop context resets entirely for Opus 4.5 in their long-running harness. Test and observe with your specific model; if quality degrades late in sessions, try context resets before blaming the task complexity.

### Multi-Session Handoffs

When a task is too large for a single session, or when context is getting full:

**At the end of each session:**
1. Commit all work-in-progress to git with clear commit messages.
2. Update the progress file with: what was completed, what's still pending, any blockers or decisions made, and explicit next steps.
3. Update the feature list: mark completed features as passing. **Never remove or edit feature descriptions — only change their status.**
4. Leave the codebase in a clean, working state. No half-implemented features. No broken builds.

**At the start of each new session:**
1. Read the agent instruction file (`CLAUDE.md`, `.cursor/rules/`, `AGENTS.md`) for repo conventions.
2. Read the progress file and feature list to understand current state.
3. Read recent git log (`git log --oneline -20`) to see what changed recently.
4. Run the setup script and a basic smoke test to verify the app works.
5. If the smoke test fails, fix the existing issue before starting new work.
6. Pick the highest-priority incomplete feature and work on it.

## Separated Evaluation

For complex or subjective tasks, separate generation from evaluation:

- **Don't self-evaluate your own work** — Agents (and humans) tend to be overly generous when judging their own output.
- **Resist verification shortcuts** — Recognize these rationalizations and do the opposite:
  - "The code looks correct based on my reading" → **Reading is not verification. Run it.**
  - "The tests already pass" → **The implementer is an LLM. Verify independently.**
  - "This is probably fine" → **Probably is not verified. Run it.**
  - "Let me check the code to verify" → **No. Start the server and hit the endpoint.**

Example verification dialogue:
```
# BAD: self-congratulatory verification
"I've reviewed the code and the login endpoint correctly validates credentials
and returns a JWT token. The implementation looks complete."

# GOOD: real verification with tools
"Started the dev server. POST /auth/login with valid credentials → 200, response
contains 'token' field (JWT, 3 segments). POST with wrong password → 401,
response body is {"error": "Invalid credentials"}. POST with missing email → 400.
Verified all three cases by actually calling the endpoint."
```
- When quality matters, use a **separate evaluation pass**:
  1. Implement the feature (generator role).
  2. Then critically test it from a user's perspective (evaluator role): use the actual UI, call the actual API, check the actual database.
  3. Grade against concrete criteria, not vibes.
  4. If criteria aren't met, iterate with specific feedback.
- For UI work, this means using browser automation to interact with the live application, take screenshots, and verify behavior end-to-end.
- For API work, this means making real HTTP requests and checking responses.
- **Calibrate the evaluator with examples** — If using a separate evaluator agent, provide few-shot examples with detailed score breakdowns in its prompt. This anchors the evaluator's judgment to your quality standards, reduces score drift across iterations, and prevents the evaluator from being too lenient or too harsh.
- **Use files for inter-agent communication** — When generator and evaluator are separate agents, communicate via files (e.g., `sprint_contract.md`, `eval_feedback.md`) rather than direct message passing. One agent writes a file, the other reads it and responds. This creates an auditable trail and works across context resets.

## Anti-Patterns

| Don't | Do Instead |
|-------|-----------|
| Rewrite working code wholesale | Extend with new code paths |
| Make multiple unrelated changes in one step | One concern per change |
| Skip reading the file before editing | Always read first |
| Assume the code structure from memory | Verify with actual file reads |
| Introduce side effects in "cleanup" | Keep cleanup separate from feature work |
| Ignore test/lint failures to "fix later" | Fix immediately before proceeding |
| Guess at runtime behavior | Trace the code path or add logging |
| Mark a feature done without running it | Test end-to-end as a user would |
| Try to one-shot an entire application | Work one feature at a time, incrementally |
| Pre-load entire codebase into context | Read files on demand, use sub-agents for exploration |
| Push through errors hoping they'll resolve | Stop, diagnose, fix or revert |
| Declare victory too early | Check the feature list — is everything passing? |
| Retry indefinitely on automated failures | Circuit-break after 3 consecutive failures |
| Say "the code looks correct" as verification | Run it. Reading is not verification |
| Use vague instructions like "be concise" | Use quantified constraints: "≤25 words" or "≤3 sentences" |
| Keep verbose old tool output in context | Collapse to conclusions once details are no longer needed |
| Ignore the repo's agent instruction file | Read CLAUDE.md / .cursor/rules / AGENTS.md first |
| Implement stubs/placeholders to get it compiling | Implement fully — no TODOs, no stubs, no "// implement later" |
| Follow a stale plan without questioning it | Compare plan against current codebase state; regenerate if drifted |
| Start with multi-agent when single-agent works | Use one agent until you hit a demonstrable ceiling, then add more |
| Rely on instructions alone for quality enforcement | Use hooks and CI for deterministic guarantees — advisory rules get ignored under pressure |
| Skip requirements clarification on vague tasks | Interview the user about constraints, edge cases, and "done" criteria before planning |

## Failure Recovery

When something goes wrong:
1. **Stop and assess** — Don't compound the error with more changes.
2. **Identify the root cause** — Read the error, re-read the modified code, trace the logic.
3. **Fix forward or revert** — If the fix is obvious and small, fix it. If unclear, revert to a known-good state and re-approach:
   - **Lightweight rewind**: Use your agent's checkpoint/rewind feature (e.g., `/rewind` in Claude Code, undo in Cursor) to restore both conversation and code state. This is faster than git revert and preserves context better.
   - **Git revert**: Use `git revert` or `git checkout` when you need to undo committed changes or the agent doesn't support checkpoints.
   - **Try something risky, then rewind**: Checkpoints make experimentation cheap — try an approach, and if it fails, rewind and try a different one rather than debugging a broken state.
4. **Never repeat a failed approach** without a new hypothesis for why it will work this time.
5. **If stuck after 3 attempts** — Stop, document the problem clearly, and ask for human guidance rather than burning context on repeated failures.
6. **Circuit breaker on automated retries** — Any retry loop (tool calls, builds, API requests) MUST have a maximum consecutive failure limit. Default to 3.
7. **Clean restart over accumulated corrections** — If you've corrected the same issue more than twice in one session, the context is cluttered with failed approaches. Clear the context and start fresh with a more specific prompt that incorporates what you learned.

## Tool Usage Principles

When using tools during execution:
- **Discover before using** — When many tools are available, examine what's accessible before starting work. Match tool choice to user intent. Don't default to general-purpose tools when specialized ones exist for the task.
- **Consolidate operations** — Prefer tools that do multiple related things in one call over making many small calls.
- **Be token-efficient** — Request concise responses when you only need IDs or status; request detailed responses only when you need full context.
- **Namespace awareness** — When many tools are available, use the most specific one for the task. Don't use a general tool when a specialized one exists.
- **Handle errors gracefully** — Tool calls can fail. Check responses before acting on them. Don't assume success.
- **Treat denials as feedback** — When a tool call is denied (permissions, sandbox, policy), treat the denial as informative output and adjust your approach. Repeated denials signal a fundamental approach problem.
- **Deduplicate reads** — If you've already read a file this turn and it hasn't been modified, don't re-read it.
- **Collapse stale output** — Large tool outputs from several turns ago that are no longer relevant should be compressed to their key conclusions.
