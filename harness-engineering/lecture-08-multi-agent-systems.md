# Lecture 8: Multi-Agent Systems

> **Feynman Concept**: How to design harnesses for systems where multiple agents work together — and why coordination is harder than it looks.

---

## Step 1: Explain It Simply

One agent is hard enough. Now imagine ten agents working on the same codebase simultaneously.

Agent A is refactoring the authentication module. Agent B is writing tests for it. Agent C is updating the API documentation. They're all working at the same time, and they all have opinions about what the code should look like.

Without careful harness design, you get:
- Agent A and B write conflicting versions of the same file
- Agent C documents an API that Agent A just renamed
- Agent B's tests fail because Agent A's refactor isn't done yet
- No one knows what "done" means for the overall task

**Multi-agent harness design** is the practice of giving each agent a well-defined role, clear boundaries, and coordination mechanisms — so they amplify each other instead of fighting each other.

### Analogy 🎭

A film production: The director (orchestrator agent) coordinates. The cinematographer films (specialist agent). The sound engineer records audio (specialist agent). They don't all make decisions simultaneously — there's a clear hierarchy, clear handoffs, and a production schedule that prevents conflicts.

---

## Step 2: Identify Gaps

| Gap | What I Said | What I'm Not Sure About |
|-----|-------------|------------------------|
| 1 | "Well-defined role" | What does role separation actually look like in practice? |
| 2 | "Coordination mechanisms" | How do agents hand off work to each other without losing context? |
| 3 | "Orchestrator vs specialist" | How do you decide what gets orchestrated vs. executed? |
| 4 | "Shared state" | How do multiple agents safely share and modify the same files? |

---

## Step 3: Fill the Gaps

### Gap 1: Role Separation in Practice

**The answer** ([Anthropic — How we built our multi-agent research system](https://www.anthropic.com/engineering/multi-agent-research-system)):

Anthropic's multi-agent research system separates agents by:
- **Scope of responsibility** — each agent owns a bounded domain (one module, one document type, one task type)
- **Access level** — agents only have tools relevant to their role
- **Output format** — each agent produces a structured artifact that the next agent consumes

Example role structure:
```
Orchestrator Agent
├── owns: overall task plan, progress tracking
├── tools: spawn_subagent, read_artifact, write_task_status
└── does NOT: write code, modify files

Implementation Agent
├── owns: writing code for assigned module
├── tools: read_file, write_file, run_tests
└── does NOT: spawn agents, modify other modules

Reviewer Agent
├── owns: quality check of implementation artifacts
├── tools: read_file, run_tests, write_review
└── does NOT: modify files, only reads and comments
```

**Simple version**: Each agent has a job description, a set of tools, and a boundary. The harness enforces the boundary.

---

### Gap 2: Coordination and Handoffs

**The question**: How do agents hand work to each other without losing context?

**The answer** ([Anthropic — Building effective agents](https://www.anthropic.com/engineering/building-effective-agents)):

Anthropic's pattern: agents communicate through **structured artifacts**, not through shared context windows.

Each agent:
1. Reads its input artifact (what was handed to it)
2. Does its work
3. Writes its output artifact (what it hands to the next agent)
4. The output artifact is the *only* channel of communication

This means:
- No shared mutable state between agents
- Each agent starts fresh with a clean context (plus the input artifact)
- Failures are isolated — one agent failing doesn't corrupt another's context
- The orchestrator can inspect any artifact to understand system state

**Simple version**: Agents talk through documents, not through shared memory. Each document is a clean, inspectable handoff.

---

### Gap 3: Orchestrator vs. Specialist

**The question**: What goes in the orchestrator and what goes in a specialist agent?

**The answer** (from Anthropic's multi-agent article):

| Decision | Orchestrator | Specialist |
|----------|-------------|-----------|
| What to do next | ✅ | ❌ |
| How to do a specific task | ❌ | ✅ |
| Whether task is done | ✅ (verification) | ✅ (self-report) |
| Tools for execution | ❌ | ✅ |
| Task planning | ✅ | ❌ |
| Error recovery | ✅ (retry/reassign) | ✅ (within task) |

The orchestrator is a **planner and coordinator**. It should not execute. Specialists are **executors**. They should not plan beyond their assigned task.

**Simple version**: Orchestrator = project manager. Specialists = engineers. Don't ask the PM to write code, don't ask engineers to manage the roadmap.

---

### Gap 4: Shared State and Conflicts

**The question**: How do you prevent agents from writing conflicting changes to the same file?

**The answer**:

Three harness patterns:

**1. Exclusive assignment** — each file/module assigned to exactly one agent at a time. The orchestrator tracks assignments. No two agents can hold the same assignment simultaneously.

**2. Artifact-first workflow** — agents write to intermediate artifacts, not directly to the codebase. A merge step (handled by the orchestrator or a dedicated merge agent) integrates changes.

**3. Git-branch isolation** — each agent works on its own branch. The orchestrator manages merges after each agent completes. Conflicts are resolved explicitly, not silently overwritten.

**Simple version**: Same as how human teams avoid merge conflicts — checkout your branch, work in isolation, merge carefully. Agents need the same discipline.

---

## Step 4: Refined Explanation

### The Three Multi-Agent Patterns

**Pattern 1: Sequential Pipeline**
```
Task → Agent A → Artifact A → Agent B → Artifact B → Agent C → Done
```
- Simplest to reason about
- Each agent has full context from the previous artifact
- Bottleneck: a failed agent blocks the whole pipeline

**Pattern 2: Parallel Fan-Out**
```
Task → Orchestrator → [Agent A, Agent B, Agent C] → Merge → Done
```
- Fastest for parallelizable work
- Requires a merge step that can integrate potentially conflicting outputs
- Best for: "each agent works on a different module"

**Pattern 3: Hierarchical (Tree)**
```
Orchestrator
├── Sub-orchestrator A → [Agent A1, Agent A2]
└── Sub-orchestrator B → [Agent B1, Agent B2]
```
- Scales to very large tasks
- Each level manages its own scope
- Complexity increases rapidly — only use when parallelism is essential

### When Multi-Agent Adds Value (and When It Doesn't)

✅ **Use multi-agent when**:
- Task is genuinely parallelizable (independent modules, independent documents)
- Each sub-task is complex enough to need its own context window
- Different roles need different tool access for security reasons

❌ **Don't use multi-agent when**:
- Tasks are sequential and interdependent
- The coordination overhead exceeds the parallelism benefit
- A single agent with good context management would suffice

**Rule of thumb**: Start with one agent. Add agents when you hit a concrete bottleneck (context overflow, parallelism need, security boundary).

### Key Takeaways

1. Agents communicate through structured artifacts, not shared context.
2. Each agent has a defined role, bounded scope, and limited tool access.
3. The orchestrator plans and coordinates — it does not execute.
4. Start single-agent. Only add agents when you have a specific reason.

---

### 30-Second Elevator Pitch

> "Multi-agent systems amplify agents — but only if each agent has clear role separation, bounded scope, and structured handoffs. Without that, agents fight each other instead of helping each other."

---

## 🧪 Practice 8

**Design exercise**: Design a multi-agent harness for this task: *"Audit a codebase for security vulnerabilities and produce a remediation plan."*

**Part A — Role design**: Define 3-4 agent roles. For each, specify:
- What it owns
- What tools it has
- What it does NOT do

**Part B — Artifact chain**: What are the structured artifacts that flow between agents? Describe the format/contents of each handoff document.

**Part C — Conflict prevention**: Two agents might both identify the same vulnerability and propose different fixes. How does your harness design prevent conflicting remediation plans?

**Part D — When to go parallel**: Which parts of this audit can run in parallel? Which parts must be sequential? Draw a simple dependency diagram.

---

## Further Reading

- [How we built our multi-agent research system](https://www.anthropic.com/engineering/multi-agent-research-system) — Anthropic's architecture with role separation and structured coordination
- [Building effective agents](https://www.anthropic.com/engineering/building-effective-agents) — Anthropic's guide to workflows, agents, and when structured systems win
- [MAgIC benchmark](https://zhiyuanhubj.github.io/MAgIC/) — Measuring cognition and collaboration in multi-agent systems
- [deepagents](https://github.com/langchain-ai/deepagents) — LangChain's toolkit for longer-running multi-agent systems

---

*← [Lecture 7: Runtimes & Reference Implementations](./lecture-07-runtimes-and-implementations.md)*
*→ [Lecture 9: Tool Design](./lecture-09-tool-design.md)*
