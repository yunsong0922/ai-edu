# Lecture 2: Context & Memory — The Agent's Working Memory

> **Feynman Concept**: How AI agents manage information across a long task, and why this is a hard engineering problem.

---

## Step 1: Explain It Simply

Imagine you're doing a complex task — rewriting an entire module in a large codebase. You start by reading files. Then more files. You write some code. You run tests. But you can only hold so much in your head at once.

AI agents have the same problem. They have a **context window** — a fixed amount of "working memory." Imagine a whiteboard that fits only 100 sticky notes. As you add notes, old ones fall off the edge.

If the wrong notes fall off — like "don't use deprecated API X" or "the user's real goal is Y" — the agent breaks things it shouldn't.

**Context engineering** is the art of deciding: *which notes go on the whiteboard, and when?*

### Analogy 🧠

A surgeon in a 12-hour operation: They can't hold everything in memory simultaneously. They rely on a scrub nurse (context management) to hand them the right tool at the right moment, track sponges (working state), and log everything (artifacts). Without that nurse, even the best surgeon makes mistakes.

---

## Step 2: Identify Gaps

| Gap | What I Said | What I'm Not Sure About |
|-----|-------------|------------------------|
| 1 | "Fixed amount of working memory" | How does KV-cache locality affect this practically? |
| 2 | "Wrong notes fall off" | How do you decide *what* to keep in context? |
| 3 | "Context drift" | What is context drift exactly, and how does it accumulate? |
| 4 | "Resuming long sessions" | How does a harness help an agent pick up where it left off? |

---

## Step 3: Fill the Gaps

### Gap 1: KV-Cache Locality

**The question**: What's the practical impact of *where* you put things in context?

**The answer** ([Manus — Context Engineering for AI Agents](https://manus.im/blog/Context-Engineering-for-AI-Agents-Lessons-from-Building-Manus)):

Inference engines cache the "key-value" computations for tokens they've seen before. If your system prompt stays identical and at the start of every request, the engine caches it and reuses it cheaply. If you shuffle content around, you break the cache and pay full compute cost every time.

**Simple version**: Keep your most-referenced stuff (system prompt, core instructions) at the TOP and never move it. Think of it like a CPU cache — stable data = fast.

---

### Gap 2: What to Keep in Context

**The question**: When the window is 80% full, what do you throw away?

**The answer** ([OpenHands — Context Condensation](https://openhands.dev/blog/openhands-context-condensensation-for-more-efficient-ai-agents)):

OpenHands' design preserves exactly four things when compressing:
1. **Goals** — what the agent is trying to accomplish
2. **Progress** — what has been done so far (summarized)
3. **Critical file paths** — the key files being modified
4. **Failing tests** — any tests that aren't passing yet

Everything else — verbose tool outputs, intermediate steps, repeated file reads — gets compressed or dropped.

**Simple version**: Keep the map, current position, and the one warning sign you must not pass. Drop the scenic description of where you've already been.

---

### Gap 3: Context Drift

**The question**: What is context drift and why does it matter?

**The answer** ([HumanLayer — Advanced Context Engineering](https://www.humanlayer.dev/blog/advanced-context-engineering)):

Context drift is the gradual accumulation of misalignment between the agent's "mental model" and reality. It's caused by:
- Noisy tool outputs filling the window with irrelevant detail
- Contradictory instructions building up over turns
- The agent "forgetting" early constraints as they scroll out of view

It compounds: each drift makes the next response slightly worse, which generates noisier output, which drifts further.

**Simple version**: Like a game of telephone — each relay slightly distorts the message. Long agent sessions accumulate drift unless the harness actively resets or re-injects ground truth.

---

### Gap 4: Resuming Long Sessions

**The question**: How does an agent pick up a multi-hour task after a context window boundary?

**The answer** ([Anthropic — Effective Harnesses](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents)):

Anthropic's pattern: before hitting the context limit, the agent creates a **handoff artifact** — a structured document summarizing:
- What was accomplished
- What remains
- Current state of files and tests
- Any open questions or blockers

The next context window (or next agent) reads this artifact to resume without having to re-read the entire history.

**Simple version**: Like a doctor writing end-of-shift notes so the night shift can continue without a full re-briefing.

---

## Step 4: Refined Explanation

### The Three Memory Problems

| Problem | What Happens | Harness Solution |
|---------|-------------|-----------------|
| **Overflow** | Context window fills up, early info lost | Summarize & compress old context |
| **Drift** | Agent "forgets" early constraints | Periodically re-inject key invariants |
| **Noise** | Useless tool outputs crowd out useful info | Filter and summarize tool results before adding to context |

### The Context Budget Mindset

Think of the context window as a **RAM budget**, not a log file:

```
Context Window (e.g. 200k tokens)
├── System prompt / agent file     [~5%]  ← NEVER MOVES (cache this)
├── Task goals + constraints        [~5%]  ← Re-inject if near limit
├── Working state (current task)   [~20%] ← Active, often updated
├── Tool results (filtered)        [~30%] ← Summarize aggressively
└── Conversation history           [~40%] ← Compress old turns
```

### Final Simple Explanation

A well-designed harness treats the context window like precious RAM:
1. Stable instructions at the top (cached cheaply by the inference engine)
2. Actively summarize old turns instead of letting them pile up
3. Re-inject critical constraints when starting a new task segment
4. Create handoff artifacts at context boundaries so sessions can resume cleanly
5. Filter noisy tool outputs before they enter context

### Key Takeaways

1. Context window = whiteboard with limited space. Manage it deliberately.
2. The harness, not the model, decides what stays and what leaves.
3. Long tasks need active memory management: summarization, compression, re-injection, handoff artifacts.

---

### 30-Second Elevator Pitch

> "Context is scarce working memory. A good harness decides what to keep, what to summarize, and what to drop — so the agent stays grounded across a long task without drifting or forgetting its original goal."

---

## 🧪 Practice 2

**Design exercise**: You're building a harness for an agent that will refactor a 10,000-line codebase over several hours.

Answer these four questions:

1. **System prompt content** — What goes in the system prompt that NEVER changes (and benefits from KV caching)?

2. **Compression policy** — After processing each file, what gets summarized and what gets dropped from context?

3. **Must-always-retain** — Name 3 things that must stay in context no matter how full the window gets.

4. **Handoff artifact** — When the context window hits 80% full, the harness needs to create a checkpoint. What does that checkpoint document contain?

> 💡 Bonus: What would context drift look like in this refactoring task? What's the first sign the agent has drifted?

---

## Further Reading

- [Effective context engineering for AI agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents) — Anthropic on managing the context window as a working memory budget
- [Context Engineering for AI Agents: Lessons from Building Manus](https://manus.im/blog/Context-Engineering-for-AI-Agents-Lessons-from-Building-Manus) — KV-cache locality, tool masking, filesystem memory
- [OpenHands Context Condensation](https://openhands.dev/blog/openhands-context-condensensation-for-more-efficient-ai-agents) — Design for bounded conversation memory
- [Advanced Context Engineering for Coding Agents](https://www.humanlayer.dev/blog/advanced-context-engineering) — Patterns for reducing context drift
- [Context-Efficient Backpressure for Coding Agents](https://www.humanlayer.dev/blog/context-efficient-backpressure) — Preventing agents from burning context on low-value work
- [Writing a good CLAUDE.md](https://www.humanlayer.dev/blog/writing-a-good-claude-md) — Practical guide to durable repo-local instructions

---

*← [Lecture 1: What Is a Harness?](./lecture-01-what-is-a-harness.md)*
*→ [Lecture 3: Constraints & Safe Autonomy](./lecture-03-constraints-and-safe-autonomy.md)*
