# Lecture 7: Runtimes & Reference Implementations

> **Feynman Concept**: What a runtime actually is, how it differs from a harness, and what you can learn by reading real open-source harness implementations.

---

## Step 1: Explain It Simply

You've designed a harness on paper — the instructions, the constraints, the evals. Now you need to actually *run* it. That's where the **runtime** comes in.

Think of it this way: a recipe is the harness (instructions, constraints, steps). The kitchen is the runtime (the environment where the recipe executes — the oven temperature, the pots, the timers). A bad kitchen can ruin a perfect recipe.

Runtimes handle the things that happen *between* the model calls:
- **State** — where is the task right now? What has been done?
- **Retries** — if a tool call fails, what happens?
- **Concurrency** — can multiple agents run in parallel?
- **Durability** — if the process crashes at step 7 of 20, can we resume?
- **Traces** — where does the log of everything go?

### Analogy 🏗️

A construction project: The blueprint (harness) tells workers what to build. The project management system (runtime) tracks who is doing what, handles delays, reassigns when someone is sick, and keeps the whole project from collapsing when one subcontractor is late.

---

## Step 2: Identify Gaps

| Gap | What I Said | What I'm Not Sure About |
|-----|-------------|------------------------|
| 1 | "Runtime handles state, retries, concurrency" | What does this look like in actual code? |
| 2 | "Reference implementations" | What can I actually learn from reading SWE-agent? |
| 3 | "Framework vs runtime vs harness" | Revisiting this distinction with concrete examples |
| 4 | "Durable execution" | What does "durable" mean and why does it matter for agents? |

---

## Step 3: Fill the Gaps

### Gap 1: What Runtime Features Look Like in Code

**The answer** ([Inngest — Your Agent Needs a Harness, Not a Framework](https://www.inngest.com/blog/your-agent-needs-a-harness-not-a-framework)):

Inngest's argument: state, retries, traces, and concurrency are *infrastructure* — they don't belong in your agent's prompt or tool logic. They belong in the runtime layer.

Concretely:

```
WITHOUT runtime infrastructure:
  agent.run(task) → if crash at step 7, start over from step 1

WITH runtime infrastructure (e.g., Inngest/AgentKit):
  agent.run(task) → state persisted after each step
                  → if crash at step 7, resume from step 7
                  → retries with exponential backoff on tool failures
                  → full trace stored regardless of outcome
```

**Simple version**: Runtime infrastructure makes your harness *survivable*. Without it, any failure means starting over.

---

### Gap 2: What You Learn from SWE-agent

**The answer** ([SWE-agent GitHub](https://github.com/SWE-agent/SWE-agent)):

SWE-agent is one of the most inspectable harness implementations available. Reading it teaches you:

1. **ACI (Agent-Computer Interface) design** — how tools are presented to the agent matters enormously. SWE-agent's tool interface is carefully designed to be simple, composable, and hard to misuse.

2. **The agent loop** — the actual while-loop that drives the agent: observe → think → act → observe → ...

3. **Environment setup** — how the benchmark environment is initialized before the agent starts (clean state, correct dependencies)

4. **Prompt engineering at scale** — how system prompts, task descriptions, and tool documentation are structured for reliability

5. **Trajectory logging** — every action stored in a structured format for eval and debugging

**Simple version**: SWE-agent is a textbook you can run. It shows you the gap between "I have an idea for a harness" and "I have a production-quality harness implementation."

---

### Gap 3: Framework vs Runtime vs Harness — Concrete Examples

| Layer | Examples | What it provides |
|-------|----------|-----------------|
| **Framework** | LangChain, LlamaIndex, CrewAI | APIs for building agents — chains, tools, memory abstractions |
| **Runtime** | Inngest, AgentKit, SWE-ReX | Execution infrastructure — state persistence, retries, durability, tracing |
| **Harness** | Your CLAUDE.md + tool config + eval suite | The *design* of what the agent does and how it's constrained |

You can build a harness without a framework (just prompts + tools). You can use a framework without a runtime (works until your first crash). The runtime is what makes harnesses production-grade.

---

### Gap 4: Durable Execution

**The question**: What does "durable" mean for agents?

**The answer** ([AgentKit — Inngest](https://github.com/inngest/agent-kit)):

Durable execution means the system guarantees that every step either completes or is retried — even if the process crashes, the network fails, or a tool times out. For long-running agents:

- A 30-minute coding task might have 50+ tool calls
- Without durability: one network timeout = 30 minutes of work lost
- With durability: one network timeout = retry that one tool call, continue from where you were

**Simple version**: Durable = the agent can be interrupted at any point and resume exactly where it left off, with no work lost.

---

## Step 4: Refined Explanation

### The Runtime Stack

```
Your Agent
    ↓
Harness Layer         ← Your CLAUDE.md, tool configs, constraints, evals
    ↓
Framework Layer       ← LangChain / raw SDK / custom loop
    ↓
Runtime Layer         ← State persistence, retries, tracing, concurrency
    ↓
Infrastructure        ← Compute, sandboxes, network, storage
```

Each layer has a job. Don't ask the harness to do the runtime's job and vice versa.

### Key Reference Implementations to Study

**[SWE-agent](https://github.com/SWE-agent/SWE-agent)** — Start here.
- Best-documented open-source coding agent harness
- Study: ACI design, agent loop, trajectory logging, environment setup
- Run it locally against a simple task and read the trace

**[SWE-ReX](https://github.com/SWE-agent/SWE-ReX)**
- Sandboxed execution infrastructure extracted from SWE-agent
- Study: How to give agents safe execution power without system access

**[AgentKit by Inngest](https://github.com/inngest/agent-kit)**
- TypeScript toolkit for durable, workflow-aware agents
- Study: How durable execution works in practice with event-driven infrastructure

**[deepagents by LangChain](https://github.com/langchain-ai/deepagents)**
- Longer-running agents with middleware and harness patterns
- Study: How middleware layers work in a real agent system

**[Harbor](https://github.com/harbor-framework/harbor)**
- Generalized harness for evaluating and improving agents at scale
- Study: How to build eval infrastructure that works across many agent configurations

### Key Takeaways

1. Runtime ≠ harness. Runtime handles infrastructure concerns (state, retries, traces). Harness handles design concerns (what to do, constraints, verification).
2. Read SWE-agent. It's the most instructive open-source harness implementation available.
3. Durable execution is not optional for production long-running agents.
4. The gap between "works in a demo" and "works reliably" is almost entirely in the runtime layer.

---

### 30-Second Elevator Pitch

> "The runtime is the kitchen your recipe runs in. Great harness design fails without durable state management, retry logic, and full tracing. Read SWE-agent to see what a production-grade implementation actually looks like."

---

## 🧪 Practice 7

**Implementation exercise**:

**Part A — Layer separation**: You're building a coding agent. For each item below, decide: does it belong in the *harness*, the *runtime*, or the *framework* layer?
- "Don't modify files in /src/legacy/"
- "If a tool call fails with timeout, retry up to 3 times"
- "Log every tool call with its parameters and result"
- "The agent should always run tests before declaring done"
- "If the process crashes, resume from the last successful step"
- "Use the Repository pattern for all database access"

**Part B — SWE-agent study**: Clone SWE-agent and run it on a simple task (or read its source). Answer:
1. What is the "agent loop" — the core while-loop that drives the agent?
2. How does it log trajectories? What format?
3. What happens when a tool call returns an error?

**Part C — Durability design**: Your agent is running a 20-step refactoring task. At step 12, the server crashes. Design a checkpoint/resume strategy:
- When does state get persisted?
- What state is persisted? (hint: what's needed to resume from step 12?)
- How does the agent "know" it's resuming vs. starting fresh?

---

## Further Reading

- [Agent Frameworks, Runtimes, and Harnesses, Oh My!](https://blog.langchain.com/agent-frameworks-runtimes-and-harnesses-oh-my/) — LangChain's decomposition
- [Your Agent Needs a Harness, Not a Framework](https://www.inngest.com/blog/your-agent-needs-a-harness-not-a-framework) — State, retries, traces as first-class infrastructure
- [SWE-agent](https://github.com/SWE-agent/SWE-agent) — Most inspectable production-quality coding harness
- [SWE-ReX](https://github.com/SWE-agent/SWE-ReX) — Sandboxed execution infrastructure
- [AgentKit](https://github.com/inngest/agent-kit) — Durable, workflow-aware agent toolkit
- [deepagents](https://github.com/langchain-ai/deepagents) — Longer-running agents with harness patterns
- [Harbor](https://github.com/harbor-framework/harbor) — Generalized harness for eval at scale
- [Building agents with the Claude Agent SDK](https://claude.com/blog/building-agents-with-the-claude-agent-sdk) — Anthropic's production-oriented SDK

---

*← [Lecture 6: Benchmarks](./lecture-06-benchmarks.md)*
*→ [Lecture 8: Multi-Agent Systems](./lecture-08-multi-agent-systems.md)*
