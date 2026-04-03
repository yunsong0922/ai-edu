# Lecture 11: Harness Design for Long-Running Applications

> **Feynman Concept**: How to build harnesses that survive across many context windows, many hours, and many failure points — for tasks that a single agent session simply can't hold.

---

## Step 1: Explain It Simply

Most agent demos show a task that fits in one session: "write this function," "fix this bug," "summarize this document."

Real software projects don't work like that. Building a full application — or even a significant feature — might take hundreds of tool calls, touch dozens of files, and span multiple days. No context window holds that.

The naive approach: just keep going until the context is full, then start over and hope the model remembers what it did.

The engineering approach: **design the harness so tasks can be paused, resumed, and handed off cleanly** — with no loss of progress and no drift from the original goal.

### Analogy 🏗️

Building a skyscraper: No single work shift builds a skyscraper. Each shift starts with a **site report** (what was done, what's in progress, what's next), works within defined zones, and ends with a **handover document** for the next shift. The harness is the project management system that makes this multi-shift work coherent.

---

## Step 2: Identify Gaps

| Gap | What I Said | What I'm Not Sure About |
|-----|-------------|------------------------|
| 1 | "Pause, resume, hand off cleanly" | What are the concrete mechanisms Anthropic uses? |
| 2 | "init.sh" | What is this and what does it do? |
| 3 | "Feature list" | How does a harness use a feature list to manage a long task? |
| 4 | "Self-verification" | How does an agent verify its own work before handing off? |

---

## Step 3: Fill the Gaps

### Gap 1: Concrete Mechanisms for Long-Running Agents

**The answer** ([Anthropic — Effective Harnesses for Long-Running Agents](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents)):

Anthropic's multi-context-window design uses three interlocking mechanisms:

**1. Initializer Agent**
A separate, lightweight agent that runs at the start of each context window. Its job:
- Read the current task state artifact
- Establish the context for the new window (what was done, what's next)
- Initialize the working environment (correct files loaded, tools configured)
- Hand off to the main implementation agent

The initializer prevents each window from "starting cold" — it bootstraps continuity.

**2. Handoff Artifacts**
At the end of each context window, the agent writes a structured document:
```markdown
# Task Handoff: Auth Module Refactor
## Completed
- Extracted UserValidator class (src/auth/UserValidator.ts)
- Updated 12 call sites (see: task/completed-sites.txt)
- Tests passing: auth.test.ts, validator.test.ts

## In Progress
- Call site update: src/api/routes/orders.ts (line 47)
- Partially updated, needs: replace validateUser() with UserValidator.validate()

## Next Steps
1. Complete orders.ts update
2. Update src/api/routes/payments.ts (similar pattern)
3. Run full test suite

## Constraints Still Active
- Do not modify legacy/ directory
- All changes must maintain backward compatibility with v1 API

## Files Modified
[list of all files touched so far]
```

**3. Orchestrator**
An external orchestrator that manages the multi-window lifecycle:
- Triggers a new context window when the current one is exhausted
- Passes the handoff artifact to the initializer
- Monitors overall progress
- Detects if an agent is looping or stuck
- Handles escalation to human review

---

### Gap 2: init.sh

**The question**: What is `init.sh` in the context of long-running agent harnesses?

**The answer** (Anthropic's harness engineering articles):

`init.sh` is a shell script that the harness runs before the agent starts work. It prepares the environment to a known state:

```bash
#!/bin/bash
# init.sh — Run before every agent session

# Ensure clean working state
git status --porcelain
if [ ! -z "$(git status --porcelain)" ]; then
  echo "WARNING: Uncommitted changes detected"
fi

# Install dependencies (idempotent)
npm install --silent

# Run baseline tests to establish starting state
npm test --silent 2>&1 | tail -5

# Validate required files exist
[ -f "TASK_STATUS.md" ] || echo "No task status found — starting fresh"
[ -f "CLAUDE.md" ] || { echo "ERROR: CLAUDE.md missing"; exit 1; }

# Report environment state
echo "Environment ready. Node: $(node --version)"
echo "Tests passing: $(npm test 2>&1 | grep 'Tests:' | tail -1)"
```

This means every agent session starts with:
- Known dependency state
- Baseline test results (so the agent knows what was already failing)
- Validated required files
- No surprise environment divergence

**Simple version**: `init.sh` is the pre-flight checklist. You run it before every session to guarantee a known starting state.

---

### Gap 3: Feature Lists as Task State

**The question**: How does a harness use a feature list to manage long tasks?

**The answer** (Anthropic long-running harness articles):

Instead of tracking progress in the agent's context (which gets lost), the harness maintains an **external feature list** — a file in the repo that tracks task state persistently:

```markdown
# feature-list.md

## Status Key
- [ ] Not started
- [~] In progress  
- [x] Complete
- [!] Blocked — needs human review

## Auth Module Refactor

### Phase 1: Extract Validator
- [x] Create UserValidator class
- [x] Write unit tests
- [x] Update auth routes

### Phase 2: Update Call Sites
- [x] src/api/routes/auth.ts
- [~] src/api/routes/orders.ts (in progress)
- [ ] src/api/routes/payments.ts
- [ ] src/api/routes/admin.ts
- [!] src/legacy/old-auth.ts (blocked: do not modify)

### Phase 3: Cleanup
- [ ] Remove deprecated validateUser() function
- [ ] Update API documentation
- [ ] Run full regression suite
```

The agent reads this file at session start (via the initializer), updates it as it works, and leaves it updated in the handoff. The orchestrator reads it to know overall progress.

**Simple version**: The feature list is a shared task board that persists across context windows. It's how the whole multi-window task stays coherent.

---

### Gap 4: Self-Verification Before Handoff

**The question**: How does an agent verify its own work before ending its context window?

**The answer** (Anthropic harness design + [Thoughtworks — Assessing internal quality](https://martinfowler.com/articles/exploring-gen-ai/ccmenu-quality.html)):

Before writing the handoff artifact, the agent runs its own verification checklist:

```
SELF-VERIFICATION PROTOCOL (run before every handoff):

1. Tests:
   - Run: npm test
   - Check: no new failures vs. baseline
   - If new failures: fix before handoff, or mark as [!] blocked

2. Scope check:
   - List all files modified
   - Verify no files outside assigned scope were touched
   - Verify no .env files, legacy/ files, etc. were touched

3. Consistency check:
   - Are there any half-finished changes? (partial refactors, commented-out code)
   - If yes: complete or revert — no half-finished work in handoff

4. Feature list update:
   - Mark completed items as [x]
   - Mark in-progress items as [~] with current status
   - Note any newly discovered blockers as [!]

5. Write handoff artifact
```

**Simple version**: The agent is its own quality gate. Before it hands off, it runs the checks a human would run.

---

## Step 4: Refined Explanation

### The Long-Running Harness Architecture

```
Session 1                    Session 2                    Session 3
┌─────────────────┐          ┌─────────────────┐          ┌─────────────────┐
│  init.sh        │          │  init.sh        │          │  init.sh        │
│  Read task state│          │  Read handoff   │          │  Read handoff   │
│  ↓              │          │  ↓              │          │  ↓              │
│  Agent works    │          │  Agent works    │          │  Agent works    │
│  ↓              │          │  ↓              │          │  ↓              │
│  Self-verify    │          │  Self-verify    │          │  Self-verify    │
│  ↓              │ ──────→  │  ↓              │ ──────→  │  ↓              │
│  Write handoff  │ handoff  │  Write handoff  │ handoff  │  Write handoff  │
└─────────────────┘ artifact └─────────────────┘ artifact └─────────────────┘
         ↑ ↑                          ↑ ↑                          ↑ ↑
         │ └── feature-list.md ───────┘ └── feature-list.md ───────┘
         └──── CLAUDE.md (always present, never modified)
```

### What Goes in Each Artifact

| Artifact | Updated by | Read by | Contains |
|----------|-----------|---------|----------|
| `CLAUDE.md` | Humans (rarely) | Every session | Permanent codebase rules |
| `feature-list.md` | Agent (every turn) | Every session, orchestrator | Current task state |
| `TASK_HANDOFF.md` | Agent (end of session) | Next session's initializer | What was done, what's next |
| `init.sh` | Humans (setup) | Harness (before each session) | Environment prep commands |

### Key Takeaways

1. Long tasks require external state — context windows don't survive across sessions.
2. Initializer agents bootstrap each new session from a known state.
3. Feature lists + handoff artifacts are the communication channel across context windows.
4. `init.sh` guarantees a clean environment before each session — no surprise state divergence.
5. Self-verification before handoff is the quality gate that prevents bad state propagation.

---

### 30-Second Elevator Pitch

> "Long-running tasks need external scaffolding: feature lists track state across sessions, handoff artifacts brief the next window, init.sh guarantees a clean start, and self-verification prevents bad state from propagating."

---

## 🧪 Practice 11

**System design exercise**: Design a long-running harness for this task:
*"Migrate a codebase from JavaScript to TypeScript — 150 files, estimated 4-6 hours of work."*

**Part A — init.sh**: Write the `init.sh` for this migration task. What should it check and report before each session?

**Part B — Feature list**: Design the `feature-list.md` structure. How do you organize 150 files across phases? What statuses do you need?

**Part C — Handoff artifact template**: Write the Markdown template for `TASK_HANDOFF.md` — what sections are required, what must always be present?

**Part D — Self-verification**: What 5 checks must the agent run before writing the handoff? Be specific to a TypeScript migration (hint: type errors, test regressions, unconverted files accidentally left as .js, etc.)

**Part E — Failure scenario**: Session 3 ends with 3 TypeScript errors the agent couldn't fix. How does the handoff handle this? What does the orchestrator do?

---

## Further Reading

- [Effective harnesses for long-running agents](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents) — Anthropic's core article on initializer agents, feature lists, init.sh
- [Harness design for long-running application development](https://www.anthropic.com/engineering/harness-design-long-running-apps) — Anthropic follow-up on task state and evaluator design
- [Assessing internal quality while coding with an agent](https://martinfowler.com/articles/exploring-gen-ai/ccmenu-quality.html) — Moving quality checks into the loop
- [EvoClaw benchmark](https://openhands.dev/blog/evoclaw-benchmark) — Evaluating agents across dependent milestone sequences from real repo history

---

*← [Lecture 10: Context Engineering Advanced](./lecture-10-context-engineering-advanced.md)*
*→ [Lecture 12: Production Harness Capstone](./lecture-12-production-harness-capstone.md)*
