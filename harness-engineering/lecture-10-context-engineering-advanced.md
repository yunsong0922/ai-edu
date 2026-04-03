# Lecture 10: Context Engineering — Advanced Patterns

> **Feynman Concept**: Beyond "manage your context window" — the engineering disciplines and concrete techniques for handling context at production scale.

---

## Step 1: Explain It Simply

Lecture 2 introduced the basics: context window is like a whiteboard, keep the important sticky notes, summarize the rest.

But when you're building a real harness for a real production workload, "summarize the rest" isn't an engineering spec. It's a wish.

Advanced context engineering asks the harder questions:
- *When exactly* do you trigger compression?
- *How* do you compress without losing critical information?
- *What happens* when an agent is generating noisy output that's filling the window faster than useful work?
- *How* do you keep the context "fresh" for a task that spans days?

### Analogy 💻

A computer's virtual memory system: When RAM fills up, the OS doesn't just randomly delete programs. It has a precise strategy — it identifies which memory pages haven't been touched recently (LRU), pages them to disk, and keeps frequently-used pages in RAM. When they're needed again, it pages them back in. Advanced context engineering is the same: deliberate, policy-driven, reversible.

---

## Step 2: Identify Gaps

| Gap | What I Said | What I'm Not Sure About |
|-----|-------------|------------------------|
| 1 | "Trigger compression at 80%" | What's the right threshold? What triggers it? |
| 2 | "Compress without losing critical info" | How do you actually compress safely? |
| 3 | "Backpressure" | What does this term mean in context engineering? |
| 4 | "Tool masking" | What is tool masking and when do you use it? |

---

## Step 3: Fill the Gaps

### Gap 1: Compression Triggers and Thresholds

**The answer** ([OpenHands — Context Condensation](https://openhands.dev/blog/openhands-context-condensensation-for-more-efficient-ai-agents)):

OpenHands' production system uses a **bounded conversation memory** design:

Compression is triggered when:
- Context reaches a threshold (e.g., 75-80% of max window)
- OR a natural breakpoint is reached (task phase boundary, tool call batch complete)

The threshold isn't about a single magic number. The key insight: trigger compression *before* you need it, not when you're at the limit. At the limit, you have no room to do the compression work itself.

**The compression policy**:
```
ALWAYS KEEP (never compress):
  - Current task goal
  - Current task constraints (from agent file)
  - List of files being modified
  - Failing test names + error messages
  - Last 2-3 turns (recent context)

SUMMARIZE (compress to ~10% of original):
  - Completed tool call sequences
  - Previous turns beyond the recent window
  - File reads of stable files (cache the summary)

DROP ENTIRELY:
  - Successful tool calls with no lasting relevance
  - Diagnostic/debug output that has been acted on
  - Repeated content (same file read multiple times)
```

**Simple version**: Compress proactively, not reactively. The last thing you want is to compress while you're in the middle of a critical reasoning step.

---

### Gap 2: Safe Compression Techniques

**The answer** ([Manus — Context Engineering for AI Agents](https://manus.im/blog/Context-Engineering-for-AI-Agents-Lessons-from-Building-Manus) + [HumanLayer — Advanced Context Engineering](https://www.humanlayer.dev/blog/advanced-context-engineering)):

**Technique 1: Progressive summarization**
Don't compress everything at once. Summarize in layers:
- Recent turns: kept verbatim
- Medium-age turns: summarized at sentence level
- Old turns: summarized at paragraph level
- Very old turns: one-line entries in a "completed work" log

**Technique 2: Persistent filesystem memory**
Move information out of context into files:
```
Instead of: keeping 500 lines of file analysis in context
Do: write analysis to /task/notes/auth-module-analysis.md
    keep in context: "Analysis written to /task/notes/auth-module-analysis.md — key finding: deprecated API on line 247"
```

The agent can re-read the file if needed, but it doesn't occupy context constantly.

**Technique 3: KV-cache locality preservation**
When compressing, keep the structure stable:
- System prompt: always at position 0, never modified
- Agent file contents: always at position 1, never modified
- Compressed history: always in the same structural slot
- Active working context: always at the end

Moving content between positions breaks the KV cache and increases cost.

**Simple version**: Compress incrementally, externalize to files when possible, and keep the structure of your context stable so the cache works.

---

### Gap 3: Backpressure

**The question**: What is backpressure in the context of agents?

**The answer** ([HumanLayer — Context-Efficient Backpressure](https://www.humanlayer.dev/blog/context-efficient-backpressure)):

In distributed systems, backpressure is the mechanism that slows down producers when consumers can't keep up. In context engineering, backpressure is the harness mechanism that prevents agents from generating more context than they can productively use.

Symptoms that need backpressure:
- Agent is doing exploratory searches that fill context without contributing to the task
- Tool calls returning large outputs that the agent reads but doesn't use
- Agent generating verbose reasoning that doesn't lead to action

Backpressure techniques:
1. **Tool output caps** — tools return at most N lines; agent must ask for more if needed
2. **Search result filtering** — don't return 100 search results; return 10 with summaries
3. **Forced checkpoints** — after N tool calls, agent must summarize progress before continuing
4. **Token budget signals** — tell the agent explicitly how much context budget remains

**Simple version**: If your agent is burning context on noise, the harness needs to slow down the noise production, not just clean up after it.

---

### Gap 4: Tool Masking

**The question**: What is tool masking?

**The answer** ([Manus — Context Engineering](https://manus.im/blog/Context-Engineering-for-AI-Agents-Lessons-from-Building-Manus)):

Tool masking is the practice of hiding tools from the agent's context when they're not relevant to the current task phase.

Example:
```
Phase: Planning
  Available tools: read_file, list_directory, search_code
  Masked tools: write_file, run_tests, deploy (hidden from context)

Phase: Implementation
  Available tools: read_file, write_file, run_tests
  Masked tools: deploy, send_email (hidden from context)

Phase: Verification
  Available tools: run_tests, read_file
  Masked tools: write_file, deploy (hidden from context)
```

Benefits:
- Reduces the context space the agent reasons over
- Prevents agents from taking premature actions (can't call what isn't visible)
- Narrows the decision space → better decisions in the available space

**Simple version**: Don't put the "deploy" button on the screen when the agent is still writing code.

---

## Step 4: Refined Explanation

### The Advanced Context Engineering Stack

```
Level 1 — Basic (Lecture 2):
  Keep important things, summarize the rest, handoff artifacts

Level 2 — Intermediate:
  Proactive compression, filesystem memory, structured slots

Level 3 — Advanced (this lecture):
  KV-cache locality, tool masking, backpressure, phase-aware context

Level 4 — Production:
  Automated compression pipelines, context quality metrics,
  A/B testing context strategies, compression eval suite
```

### Context as a First-Class Resource

Production harnesses treat context like money — every token has a cost, and you budget deliberately:

```
Context Budget: 200k tokens

Fixed costs (non-negotiable):
  System prompt:           3k tokens
  Agent file (CLAUDE.md):  2k tokens
  Current task goal:       0.5k tokens
  Total fixed:             5.5k tokens

Variable budget: 194.5k tokens remaining

Allocation:
  Active working context:  40k  (20%)
  Tool results (filtered): 60k  (30%)
  Compressed history:      94.5k (50%)
```

When you approach a budget limit, you compress the most compressible thing first (old history), not the most important thing.

### Key Takeaways

1. Compress proactively at 70-80%, not reactively at 100%.
2. Externalize stable information to files — context is for active reasoning, not storage.
3. Tool masking limits the decision space — agents make better decisions with fewer irrelevant options.
4. Backpressure prevents context flooding — slow down noise production at the source.

---

### 30-Second Elevator Pitch

> "Advanced context engineering is cache management for AI. Budget deliberately, compress proactively, externalize to files, mask irrelevant tools, and apply backpressure before the window floods."

---

## 🧪 Practice 10

**Engineering exercise**:

**Part A — Compression policy**: Write a concrete compression policy for an agent that's refactoring a large codebase. Be specific:
- What triggers compression (threshold + conditions)?
- What is ALWAYS kept verbatim?
- What is summarized (and how — one sentence per X lines)?
- What is dropped entirely?

**Part B — Backpressure scenario**: Your agent is searching the codebase for usages of a deprecated function. It runs 15 search calls, each returning 200 lines of results. By the time it's done searching, it's used 40% of the context window on search results it probably won't re-read.

Design a backpressure mechanism: How would you change the tool or the harness to prevent this?

**Part C — Tool masking design**: Design a 3-phase tool masking strategy for a "write a new feature" task:
- Phase 1: Understanding (reading and planning)
- Phase 2: Implementation (writing code)
- Phase 3: Verification (testing and review)

For each phase, list which tools are visible and which are masked.

---

## Further Reading

- [Effective context engineering for AI agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents) — Anthropic on the context window as working memory budget
- [Context Engineering for AI Agents: Lessons from Building Manus](https://manus.im/blog/Context-Engineering-for-AI-Agents-Lessons-from-Building-Manus) — KV-cache locality, tool masking, filesystem memory
- [OpenHands Context Condensation](https://openhands.dev/blog/openhands-context-condensensation-for-more-efficient-ai-agents) — Bounded conversation memory design
- [Advanced Context Engineering for Coding Agents](https://www.humanlayer.dev/blog/advanced-context-engineering) — Reducing context drift
- [Context-Efficient Backpressure for Coding Agents](https://www.humanlayer.dev/blog/context-efficient-backpressure) — Preventing low-value context consumption
- [Context Engineering for Coding Agents (Thoughtworks)](https://martinfowler.com/articles/exploring-gen-ai/context-engineering-coding-agents.html) — Shaping the task environment

---

*← [Lecture 9: Tool Design](./lecture-09-tool-design.md)*
*→ [Lecture 11: Long-Running Apps](./lecture-11-long-running-apps.md)*
