# Lecture 5: Evals & Observability — How Do You Know It Worked?

> **Feynman Concept**: How to measure whether your agent actually did a good job, and how to see inside its decision-making.

---

## Step 1: Explain It Simply

You send an agent to fix 10 bugs. It comes back and says "done!"

Did it actually fix them? Did it break 5 other things in the process? Did it take an unnecessarily expensive path — reading 50 files when 5 would have been enough? Did it hallucinate a fix that compiles but is logically wrong?

**You don't know — unless you built evaluation into your harness.**

**Evals** are tests for agent behavior. Not just "does the code compile" — but: did the agent follow the right trajectory? Did it use the right tools? Did it handle the ambiguous case correctly?

**Observability** is logging and tracing everything the agent does so you can understand *why* it succeeded or failed — and improve the harness systematically.

### Analogy 🏥

Doctor doing a health checkup: You don't just ask "do you feel okay?" You run tests — blood pressure, cholesterol, reflexes. Each test targets a specific failure mode. Agent evals are the same — each eval targets a specific reliability failure mode in your harness.

---

## Step 2: Identify Gaps

| Gap | What I Said | What I'm Not Sure About |
|-----|-------------|------------------------|
| 1 | "Evals are tests for agent behavior" | How are agent evals different from regular unit tests? |
| 2 | "Did it follow the right trajectory?" | What is trajectory-level evaluation? |
| 3 | "Infrastructure noise" | What does this mean and why should I care? |
| 4 | "Observability" | What specifically should I log for an agent? |

---

## Step 3: Fill the Gaps

### Gap 1: Agent Evals vs Unit Tests

**The question**: My codebase already has unit tests. Why do I need evals?

**The answer** ([OpenAI — Testing Agent Skills Systematically](https://developers.openai.com/blog/eval-skills/) + [OpenHands — How to Evaluate Agent Skills](https://openhands.dev/blog/evaluating-agent-skills)):

Unit tests check *output correctness* of a specific function.
Agent evals check *behavioral correctness* of the agent across a task:

| Dimension | Unit Test | Agent Eval |
|-----------|-----------|-----------|
| What it tests | One function's output | Agent's task completion |
| Input | Deterministic inputs | Realistic task descriptions |
| Success criterion | Return value matches expected | Task goal achieved (many valid paths) |
| Non-determinism | None | High (agent can succeed many ways) |
| What it reveals | Code bugs | Harness gaps, skill gaps, reasoning failures |

Agent evals need **deterministic verifiers** — automated checks that confirm the task goal was achieved, independent of which path the agent took.

**Simple version**: Unit tests say "this function returns 42." Evals say "the agent correctly handled the ambiguous user request and produced working, tested code."

---

### Gap 2: Trajectory-Level Evaluation

**The question**: What does it mean to evaluate the *trajectory*, not just the final result?

**The answer** ([OpenAI — Trace Grading](https://platform.openai.com/docs/guides/trace-grading) + [LangChain — Evaluating Deep Agents](https://blog.langchain.com/evaluating-deep-agents-our-learnings/)):

A trajectory is the full sequence of actions the agent took: which tools it called, in what order, with what parameters, and what it did with the results.

Two agents can both pass the final test ("code works") but have very different trajectories:
- Agent A: read 3 files → wrote the fix → ran tests → done
- Agent B: read 50 files → rewrote 8 files → broke 2 tests → fixed them → done

Same outcome. But Agent B is 10x more expensive, takes 5x longer, and introduces unnecessary churn.

Trajectory-level evaluation catches:
- **Inefficiency** — using 10 tool calls where 2 would suffice
- **Wrong reasoning** — arriving at the right answer through invalid logic (will fail on edge cases)
- **Harness-exploiting behavior** — finding unexpected shortcuts that don't generalize

LangChain identifies three evaluation levels:
1. **Single-step** — did this tool call produce the right result?
2. **Full-run** — did the agent complete the task?
3. **Multi-turn** — did the agent maintain coherent behavior across a long session?

**Simple version**: The final answer isn't the only thing that matters. *How* the agent got there reveals whether it understood the problem or just got lucky.

---

### Gap 3: Infrastructure Noise

**The question**: What is "infrastructure noise" in evals and why does it matter?

**The answer** ([Anthropic — Quantifying infrastructure noise](https://www.anthropic.com/engineering/infrastructure-noise)):

Anthropic found that *runtime configuration alone* — things like temperature settings, tool timeout values, sandbox configurations — can move benchmark scores by more than many model-tier differences on leaderboards.

This means: if your eval environment differs from your production environment, your eval results are misleading. You might be measuring the harness, not the model. Or worse — measuring environment artifacts, not either.

**Simple version**: If your eval runs in a fast sandbox but production runs on a slow network with timeouts, your eval scores are fiction. Match the eval environment to production.

---

### Gap 4: What to Log for Observability

**The question**: What specifically should I instrument in an agent harness?

**The answer** ([Anthropic — Demystifying Evals for AI Agents](https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents) + [OpenHands — Learning to Verify AI-Generated Code](https://openhands.dev/blog/20260305-learning-to-verify-ai-generated-code)):

An agent trace should capture:

```
For each turn in the agent loop:
  - timestamp
  - input (what the agent was given)
  - reasoning (if available)
  - tool_calls: [{name, params, result, latency_ms}]
  - output (what the agent produced)
  - token_count (input + output)
  - cost_usd

For the full task:
  - task_id
  - task_description
  - success: boolean
  - success_criteria_met: [list of checks]
  - total_turns
  - total_tokens
  - total_cost_usd
  - wall_clock_time_s
  - harness_version (!)
```

The `harness_version` field is critical — when you improve the harness, you need to know which traces were produced by which version.

**Simple version**: Log everything the agent does, what it costs, and whether it succeeded. Tag each trace with your harness version so you can measure improvements over time.

---

## Step 4: Refined Explanation

### The Four Questions Every Eval Must Answer

| Question | Eval Type | Example Check |
|----------|-----------|---------------|
| Did it complete the task? | Outcome | Do the tests pass? Does the feature work? |
| Did it complete it correctly? | Correctness | Are there no regressions? Is the logic sound? |
| Did it complete it efficiently? | Trajectory | How many tool calls? How many tokens? |
| Would it work on harder cases? | Generalization | Run against 10 variants of the task |

### The Eval Pyramid

```
             ▲
            / \
           / E \ ← E2E / Full-run evals (expensive, slow, catch harness-level failures)
          /─────\
         / Multi \ ← Multi-turn evals (medium cost, catch drift and coherence failures)
        /─────────\
       / Unit/Step  \ ← Single-step evals (cheap, fast, catch tool-call correctness)
      /─────────────\
```

Run the cheap ones continuously. Run the expensive ones on significant harness changes.

### A Minimal Eval Setup

```python
# Pseudocode for a minimal agent eval

def eval_harness(agent, task, expected_outcome):
    trace = []
    result = agent.run(
        task=task,
        on_tool_call=lambda call: trace.append(call)
    )
    
    checks = {
        "task_completed": verify_outcome(result, expected_outcome),
        "no_regressions": run_test_suite(),
        "tool_calls_within_budget": len(trace) < MAX_TOOL_CALLS,
        "token_cost_within_budget": sum(t.tokens for t in trace) < MAX_TOKENS,
    }
    
    return EvalResult(
        success=all(checks.values()),
        checks=checks,
        trace=trace,
        harness_version=HARNESS_VERSION
    )
```

### Key Takeaways

1. Evals measure harness quality, not just model quality. Same model, better harness = better eval scores.
2. Trajectory evaluation catches problems that outcome evaluation misses.
3. Match your eval environment to production — infrastructure noise can be larger than model differences.
4. Tag every trace with a harness version. That's how you measure progress.

---

### 30-Second Elevator Pitch

> "Evals answer: did it work, and is it getting better? Without them, you're flying blind — you can't distinguish a harness problem from a model problem, and you can't measure improvement."

---

## 🧪 Practice 5

**Design exercise**: Design an eval suite for the PR review agent from Practice 1.

**Part A — Outcome Eval**: Write 3 concrete, verifiable success criteria for "the agent reviewed this PR correctly."  
(Hint: each criterion must be checkable by a script, not just a human judgment.)

**Part B — Trajectory Eval**: What would a "bad trajectory" look like even if the outcome is correct?  
(Hint: think about tool call count, which files it read, token cost.)

**Part C — Eval Dataset**: List 5 different PR scenarios you'd include in your eval set to make it robust.  
(Hint: think about edge cases — empty PRs, huge PRs, PRs that break tests, PRs touching sensitive files.)

**Part D — Infrastructure Match**: Your agent runs in a GitHub Actions sandbox for evals. List 2 ways the eval environment might differ from production and invalidate your eval scores.

---

## Final Integration Exercise: Putting It All Together

You've now covered all 5 pillars. Design a **complete harness** for this scenario:

> **Task**: Build an agent that monitors your GitHub repo for new issues, triages them (assigns labels, priority), and drafts a first response.

For each pillar, write 2-3 sentences:

| Pillar | Your Harness Design |
|--------|-------------------|
| **Lecture 1: What is the harness?** | What components make up this harness? |
| **Lecture 2: Context & Memory** | What stays in context? What gets summarized? What's the handoff artifact? |
| **Lecture 3: Constraints & Safe Autonomy** | What can it do autonomously? What requires a checkpoint? What is forbidden? |
| **Lecture 4: Specs & Agent Files** | What goes in the `CLAUDE.md`? What spec does each issue get? |
| **Lecture 5: Evals & Observability** | How do you know a triage was correct? What do you log? |

---

## Further Reading

- [Testing Agent Skills Systematically with Evals](https://developers.openai.com/blog/eval-skills/) — OpenAI: turning traces into repeatable evals
- [How to Evaluate Agent Skills](https://openhands.dev/blog/evaluating-agent-skills) — OpenHands: bounded tasks, deterministic verifiers, trace review
- [Demystifying Evals for AI Agents](https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents) — Anthropic: what to measure when agents have many paths to success
- [Quantifying infrastructure noise in agentic coding evals](https://www.anthropic.com/engineering/infrastructure-noise) — Anthropic: eval environment matters
- [Evaluating Deep Agents: Our Learnings](https://blog.langchain.com/evaluating-deep-agents-our-learnings/) — LangChain: single-step, full-run, and multi-turn eval design
- [Improving Deep Agents with harness engineering](https://blog.langchain.com/improving-deep-agents-with-harness-engineering/) — LangChain: evidence that harness changes improve benchmark performance
- [Trace grading](https://platform.openai.com/docs/guides/trace-grading) — OpenAI: grading agent traces directly
- [Learning to Verify AI-Generated Code](https://openhands.dev/blog/20260305-learning-to-verify-ai-generated-code) — OpenHands: layered verification using trajectory critics

---

*← [Lecture 4: Specs, Agent Files & Workflow Design](./lecture-04-specs-agent-files-workflow.md)*
*→ Back to [Course Overview](./README.md)*
