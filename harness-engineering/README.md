# 🎓 Harness Engineering: A Feynman Lecture Series

> Based on: [walkinglabs/awesome-harness-engineering](https://github.com/walkinglabs/awesome-harness-engineering)

This is a self-paced course on **Harness Engineering** — the practice of shaping the environment around AI agents so they work reliably.

Using the **Feynman Technique**: explain simply → find gaps → fill gaps → explain better.

---

## What is Harness Engineering?

When you give an AI agent a task like "build me this app," the *model* is only half the equation. The other half is everything *around* the model: the instructions, the memory, the tools, the guardrails, the checkpoints.

That "everything around" is the **harness**.

> Harness engineering sits at the intersection of context engineering, evaluation, observability, orchestration, safe autonomy, and software architecture.

---

**The central insight**: Better harnesses beat better models — consistently, measurably, and repeatably.

---

## Course Structure (12 Lectures)

| # | File | Topic | Analogy | Core Insight |
|---|------|--------|---------|-------------|
| 1 | [lecture-01](./lecture-01-what-is-a-harness.md) | What is a Harness? | 🐕 Training a dog | Model is one component; harness is everything else |
| 2 | [lecture-02](./lecture-02-context-and-memory.md) | Context & Memory | 🧠 Whiteboard with sticky notes | Context window = RAM; manage it deliberately |
| 3 | [lecture-03](./lecture-03-constraints-and-safe-autonomy.md) | Constraints & Safe Autonomy | 🚗 Mountain road guardrails | Guardrails enable speed; constraints aren't restrictions |
| 4 | [lecture-04](./lecture-04-specs-agent-files-workflow.md) | Specs, Agent Files & Workflow | 📋 Employee handbook | Persistent instructions; CLAUDE.md is the onboarding doc |
| 5 | [lecture-05](./lecture-05-evals-and-observability.md) | Evals & Observability | 🏥 Doctor's checkup | Measure harness quality, not just model quality |
| 6 | [lecture-06](./lecture-06-benchmarks.md) | Benchmarks | 🏁 Race car design vs. engine | Same model, better harness = better benchmark scores |
| 7 | [lecture-07](./lecture-07-runtimes-and-implementations.md) | Runtimes & Reference Implementations | 🏗️ Kitchen vs. recipe | Durable runtime is what makes harnesses production-grade |
| 8 | [lecture-08](./lecture-08-multi-agent-systems.md) | Multi-Agent Systems | 🎭 Film production crew | Structured artifacts + role separation = coordination |
| 9 | [lecture-09](./lecture-09-tool-design.md) | Tool Design | 🔧 Well-designed API | One tool, one action; bad interfaces cause systematic errors |
| 10 | [lecture-10](./lecture-10-context-engineering-advanced.md) | Context Engineering Advanced | 💻 Virtual memory management | Compress proactively, mask tools, apply backpressure |
| 11 | [lecture-11](./lecture-11-long-running-apps.md) | Long-Running Applications | 🏗️ Skyscraper construction shifts | init.sh + feature lists + handoff artifacts = coherence |
| 12 | [lecture-12](./lecture-12-production-harness-capstone.md) | Production Harness Capstone | 🏭 Factory production line | All pillars integrated; maturity model to assess progress |

---

## The Harness Maturity Model

| Level | What Exists | Outcome |
|-------|-------------|---------|
| 0 — None | Raw model API calls | Inconsistent, unreliable |
| 1 — Basic Instructions | CLAUDE.md + basic tools | More consistent |
| 2 — Context-Managed | Compression + handoff artifacts | Long tasks viable |
| 3 — Evaluated | Evals + trace logging + self-verification | Quality measurable |
| 4 — Controlled Autonomy | Permission matrix + sandboxing + human checkpoints | Safe for real side effects |
| 5 — Production-Grade | All of above + durable execution + multi-agent + benchmarks | Reliable at scale |

---

## How to Use This Course

Each lecture follows the **Feynman format**:

1. **Step 1 — Explain Simply**: Plain language. No jargon. Bright 12-year-old can follow.
2. **Step 2 — Identify Gaps**: Honest table of what's fuzzy or unknown.
3. **Step 3 — Fill the Gaps**: Research answers. Cite primary sources.
4. **Step 4 — Refined Explanation**: The improved version with gaps filled.
5. **🧪 Practice**: A hands-on exercise to test real understanding.

---

## Key Source Material

- [OpenAI — Harness engineering: leveraging Codex in an agent-first world](https://openai.com/index/harness-engineering/)
- [Anthropic — Effective harnesses for long-running agents](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents)
- [Anthropic — Building effective agents](https://www.anthropic.com/engineering/building-effective-agents)
- [HumanLayer — Skill Issue: Harness Engineering for Coding Agents](https://www.humanlayer.dev/blog/skill-issue-harness-engineering-for-coding-agents)
- [LangChain — The Anatomy of an Agent Harness](https://blog.langchain.com/the-anatomy-of-an-agent-harness/)
- [LangChain — Improving Deep Agents with harness engineering](https://blog.langchain.com/improving-deep-agents-with-harness-engineering/)
- [Thoughtworks — Harness Engineering](https://martinfowler.com/articles/exploring-gen-ai/harness-engineering.html)
- [HumanLayer — 12 Factor Agents](https://www.humanlayer.dev/blog/12-factor-agents)
- [Manus — Context Engineering for AI Agents](https://manus.im/blog/Context-Engineering-for-AI-Agents-Lessons-from-Building-Manus)
