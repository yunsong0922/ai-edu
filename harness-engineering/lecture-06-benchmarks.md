# Lecture 6: Benchmarks — Measuring Harness Quality, Not Just Model Quality

> **Feynman Concept**: How to use standardized benchmarks to compare harnesses objectively, and why most leaderboards measure the wrong thing.

---

## Step 1: Explain It Simply

Imagine two race car teams. Both use the same engine (same model). But Team A finishes the race, and Team B crashes on lap 3.

You could look at the race result and conclude "Team A's engine is better." But that's wrong — the engines are identical. The difference is the *car design, tires, pit crew strategy, and track preparation* — the harness.

Most AI benchmarks today measure the wrong thing: they measure which model + harness combination wins, then attribute the win to the model. This is like attributing every race result to the engine alone.

**Harness-aware benchmarking** uses benchmarks to answer a different question: *given the same model, does this harness produce better outcomes?*

### Analogy 🏁

Olympic sports drug testing: The benchmark (race result) is objective. But to know if an athlete improved because of training (harness) vs. drugs (model upgrade), you need controlled comparisons. Same athlete, different training regimes = harness experiment. Different athlete = model experiment.

---

## Step 2: Identify Gaps

| Gap | What I Said | What I'm Not Sure About |
|-----|-------------|------------------------|
| 1 | "Benchmarks measure the wrong thing" | Which benchmarks are actually good for harness evaluation? |
| 2 | "Harness-aware benchmarking" | How do you actually control for model vs harness in a benchmark? |
| 3 | "SWE-bench, OSWorld, etc." | What does each benchmark actually test? |
| 4 | "Infrastructure noise" | We touched on this in Lecture 5 — how big is this effect? |

---

## Step 3: Fill the Gaps

### Gap 1: Which Benchmarks Are Good for Harness Evaluation?

**The answer**: The repo identifies benchmarks that "stress context handling, tool calling, environment control, verification logic, and runtime scaffolding around the model" — i.e., harness-sensitive benchmarks.

The best harness-evaluation benchmarks have these properties:
1. **Execution-based verifiers** — success is determined by running the output, not judging it
2. **Multi-step tasks** — single-step tasks don't expose harness strengths/weaknesses
3. **Realistic environments** — agents work in real OSes, browsers, or codebases
4. **Long-horizon** — tasks that take many turns, exposing drift and memory failures

Top harness-sensitive benchmarks from the repo:

| Benchmark | What it tests | Why it's harness-sensitive |
|-----------|--------------|---------------------------|
| [SWE-bench Verified](https://www.swebench.com/) | Fix real GitHub issues with tests | Retrieval, patching, test verification all visible |
| [OSWorld](https://os-world.github.io/) | 369 real computer tasks (Ubuntu/Win/Mac) | Desktop + multimodal harness depth |
| [Terminal-Bench](https://www.tbench.ai/) | Terminal-native agents (shells, filesystems) | Coding-agent harness design |
| [AppWorld](https://appworld.dev/) | Interactive coding agents on app tasks | Planning + code gen + collateral-damage control |
| [AgentBench](https://github.com/THUDM/AgentBench) | OS, databases, web, knowledge graphs | Harness generalization across environments |
| [WebArena](https://webarena.dev/) | Autonomous agents on realistic web tasks | Web-facing harness design |

---

### Gap 2: How to Control for Model vs. Harness

**The question**: If I improve my SWE-bench score, how do I know if it's the model or the harness?

**The answer** ([LangChain — Improving Deep Agents with harness engineering](https://blog.langchain.com/improving-deep-agents-with-harness-engineering/)):

LangChain ran a controlled experiment: same model, different harness configurations. Harness changes alone produced significant benchmark improvements — demonstrating harness quality is independently measurable.

The protocol:
1. **Fix the model** — don't change it between experiments
2. **Change one harness variable at a time** — e.g., only change the context compression strategy
3. **Run against the same benchmark version** — reproducible task set
4. **Compare traces** — not just scores, but tool call counts, token usage, failure patterns

**Simple version**: Scientific method applies. Control variables. Change one thing. Measure.

---

### Gap 3: What Each Benchmark Actually Tests

**SWE-bench Verified** 🔧
- Tasks: Fix real bugs from real GitHub repos, tests already written
- Harness reveals: Code retrieval strategy, patch application, test-running harness
- Why verified: Human-validated to remove ambiguous tasks

**OSWorld** 💻
- Tasks: 369 real computer tasks across Ubuntu, Windows, macOS
- Harness reveals: Desktop agent harness depth, screenshot/GUI understanding, multi-app workflows
- Special: Each task has initial-state setup and execution-based evaluators

**Terminal-Bench** ⌨️
- Tasks: Terminal-native agents in shells and filesystems
- Harness reveals: How the coding-agent harness handles real terminal environments
- Note: v2.0 + Harbor is a generalized evaluation harness itself

**AppWorld** 📱
- Tasks: Agents complete tasks in a "controllable world of apps and people"
- Harness reveals: Planning quality, code generation, collateral-damage control (doesn't break other things)

**AgentBench** 🌐
- Tasks: OS, databases, knowledge graphs, web browsing across environments
- Harness reveals: Whether your harness generalizes beyond one narrow task loop

---

### Gap 4: Infrastructure Noise Scale

**The question**: How much does infrastructure noise actually affect scores?

**The answer** ([Anthropic — Quantifying infrastructure noise](https://www.anthropic.com/engineering/infrastructure-noise)):

Anthropic found that runtime configuration changes could shift coding benchmark scores by amounts larger than many model-tier gaps seen on public leaderboards. This means:

- A harness optimized for a specific benchmark environment can outperform a "better" model in a worse environment
- Leaderboard comparisons are often apples-to-oranges if harness configurations differ
- Your internal benchmark environment must match production *exactly*

**Simple version**: The "noise floor" from infrastructure is higher than most people think. Don't trust leaderboard gaps smaller than your infrastructure noise.

---

## Step 4: Refined Explanation

### How to Use Benchmarks for Harness Development

**Step 1 — Choose a harness-sensitive benchmark** matching your use case:
- Coding tasks → SWE-bench Verified or Terminal-Bench
- Computer use → OSWorld
- Web tasks → WebArena or AssistantBench
- Multi-environment → AgentBench

**Step 2 — Establish a baseline** (fixed model, current harness)

**Step 3 — Iterate on one harness variable at a time**:
```
Experiment 1: change context compression strategy → re-run → compare
Experiment 2: change tool call budget → re-run → compare
Experiment 3: add self-verification step → re-run → compare
```

**Step 4 — Track trace quality**, not just scores:
- Did task completion go up?
- Did token cost go down?
- Did failure modes shift? (new failures often reveal harness gaps)

**Step 5 — Match eval environment to production**

### The Benchmark Hierarchy for Harness Work

```
Use for: development iteration (fast, cheap)
→ Single-environment benchmarks: Terminal-Bench, SWE-bench subset

Use for: harness validation (medium cost)
→ Multi-environment: AgentBench, WebArena

Use for: production readiness (expensive, comprehensive)
→ Full OSWorld, full SWE-bench Verified, AppWorld
```

### Key Takeaways

1. Most benchmark scores conflate model quality and harness quality. Separate them experimentally.
2. Choose benchmarks that expose your actual harness failure modes (long-horizon, tool-use, multi-environment).
3. Infrastructure noise can dwarf model differences — match your eval environment to production.
4. Use benchmarks iteratively: fix model, change one harness variable, measure.

---

### 30-Second Elevator Pitch

> "Benchmarks tell you if your harness is getting better — but only if you fix the model and change one harness variable at a time. Most leaderboard comparisons are meaningless because they don't control for harness differences."

---

## 🧪 Practice 6

**Benchmark analysis exercise**:

**Part A — Select the right benchmark**: You're building a harness for an agent that browses internal company wikis to answer employee questions. Which benchmark from the list above is closest to your use case? Why?

**Part B — Controlled experiment design**: You want to test whether adding a "context compression step" to your harness improves performance. Write out the experiment:
- What stays fixed?
- What changes?
- What metric(s) do you measure?
- What would "success" look like?

**Part C — Infrastructure noise audit**: List 3 ways your development environment might differ from production that could invalidate your benchmark results.

---

## Further Reading

- [SWE-bench Verified](https://www.swebench.com/) — Software engineering agent benchmark
- [OSWorld](https://os-world.github.io/) — Real computer tasks benchmark
- [AgentBench](https://github.com/THUDM/AgentBench) — Cross-environment benchmark
- [Terminal-Bench](https://www.tbench.ai/) — Terminal-native agent benchmark
- [AppWorld](https://appworld.dev/) — Interactive coding agent benchmark
- [Improving Deep Agents with harness engineering](https://blog.langchain.com/improving-deep-agents-with-harness-engineering/) — Harness changes improve benchmark scores
- [Quantifying infrastructure noise in agentic coding evals](https://www.anthropic.com/engineering/infrastructure-noise) — Infrastructure noise scale

---

*← [Lecture 5: Evals & Observability](./lecture-05-evals-and-observability.md)*
*→ [Lecture 7: Runtimes & Reference Implementations](./lecture-07-runtimes-and-implementations.md)*
