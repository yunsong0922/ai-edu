# Lecture 12: Production Harness Capstone — Putting It All Together

> **Feynman Concept**: How all 11 lectures combine into a coherent, production-grade harness design — and what separates systems that work in demos from systems that work in production.

---

## Step 1: Explain It Simply

We've covered 11 distinct pillars. Each one is important on its own. But the real skill of harness engineering isn't knowing each pillar — it's knowing how they interact, where they conflict, and how to make tradeoffs.

A production harness is not a checklist. It's a system where every component depends on and constrains every other component. Get one wrong and the others can't compensate.

This lecture synthesizes everything into a single coherent framework — and shows what "production-grade" actually means.

### Analogy 🏭

A factory production line: Each station (welding, painting, assembly, quality control) works fine in isolation. But the factory only works when the stations are correctly sequenced, have the right throughput, handle exceptions consistently, and are designed to surface — not hide — failures.

---

## Step 2: The 12-Factor Harness

*A synthesis of HumanLayer's [12-Factor Agents](https://www.humanlayer.dev/blog/12-factor-agents) and [12-Factor AgentOps](https://www.12factoragentops.com/), applied across all 11 lectures.*

| Factor | Principle | Lecture |
|--------|-----------|---------|
| **I. Single Responsibility** | Each component does one thing | L1, L8, L9 |
| **II. Explicit Instructions** | All agent guidance is visible and auditable | L4 |
| **III. Context as Resource** | Context is budgeted deliberately, not dumped | L2, L10 |
| **IV. Stateless Agents** | Agents are stateless; the harness owns state | L7, L11 |
| **V. Graduated Autonomy** | Actions are tiered by risk; high-risk requires human gate | L3 |
| **VI. Minimum Footprint** | Agents request only what they need for the current task | L3 |
| **VII. Explicit Tools** | Tools are inspectable, minimal, and well-typed | L9 |
| **VIII. Verified Output** | Agents self-verify before declaring done | L5, L11 |
| **IX. Observable** | Every action is logged, traceable, and attributed to a harness version | L5, L7 |
| **X. Resumable** | Tasks can be interrupted and resumed without progress loss | L7, L11 |
| **XI. Measurable** | Quality is tracked via evals, not intuition | L5, L6 |
| **XII. Evolvable** | The harness improves based on evidence, not guesses | L5, L6 |

---

## Step 3: What Separates Demo from Production

### The Demo/Production Gap

| Dimension | Demo | Production |
|-----------|------|-----------|
| Task length | Single session | Multi-session, multi-day |
| Error handling | Crashes are acceptable | Errors must be recovered, not just logged |
| Infrastructure | Developer's laptop | Sandboxed, isolated, resource-limited |
| Observability | Print statements | Structured traces with harness versioning |
| Evals | "It looked right" | Automated, deterministic, version-tracked |
| State management | Implicit (in context) | Explicit (external artifacts) |
| Concurrency | One agent at a time | Multiple agents, parallel workstreams |
| Human oversight | Watching every step | Checkpoints at risk boundaries only |
| Recovery | Restart from scratch | Resume from last checkpoint |
| Improvement | "Try different prompts" | Controlled harness experiments with metrics |

The gap isn't model quality. It's engineering discipline around every non-model component.

---

## Step 4: A Reference Production Harness

### Repository Structure

```
my-project/
├── CLAUDE.md                  ← Permanent codebase rules (L4)
├── init.sh                    ← Environment preparation (L11)
├── feature-list.md            ← Current task state (L11)
│
├── .harness/
│   ├── tools/                 ← Tool definitions and schemas (L9)
│   ├── evals/                 ← Eval suite (L5, L6)
│   ├── prompts/               ← System prompts (versioned) (L4)
│   └── traces/                ← Logged agent traces (L5, L7)
│
└── src/                       ← Your actual code
```

### The Agent Loop (Production-Grade)

```python
# Pseudocode for a production agent loop

def run_agent_session(task: Task, harness_version: str):
    # 1. Environment preparation (L11)
    run_init_sh()
    
    # 2. Load persistent state (L11)
    task_state = read_feature_list()
    handoff = read_last_handoff()
    
    # 3. Bootstrap context (L2, L10)
    context = ContextManager(
        budget_tokens=200_000,
        always_keep=["task_goal", "constraints", "current_phase"],
        compress_at=0.75,
        evict_policy="lru_except_always_keep"
    )
    context.load_system_prompt()         # KV-cached position (L10)
    context.load_agent_file("CLAUDE.md") # KV-cached position (L10)
    context.load_task_state(task_state)
    context.load_handoff(handoff)
    
    # 4. Configure tools for current phase (L9, L10)
    phase = task_state.current_phase
    tools = ToolRegistry.for_phase(phase)  # Tool masking (L10)
    
    # 5. Run agent loop
    trace = []
    while not task_state.complete:
        response = model.call(
            context=context.current(),
            tools=tools.available()
        )
        
        # Log every action (L5)
        trace.append({
            "turn": len(trace),
            "response": response,
            "harness_version": harness_version
        })
        
        for tool_call in response.tool_calls:
            # Authorization check (L3)
            if not tools.is_authorized(tool_call, phase):
                raise HarnessViolation(f"Unauthorized: {tool_call.name} in {phase}")
            
            # Execute in sandbox if needed (L3, L9)
            result = tools.execute_sandboxed(tool_call)
            
            # Backpressure: filter output before adding to context (L10)
            filtered_result = context.filter_tool_output(result)
            context.add_tool_result(filtered_result)
            
            # Update external state (L11)
            task_state.update_from_tool(tool_call, result)
        
        # Compress context if needed (L2, L10)
        if context.usage_ratio > 0.75:
            context.compress(strategy="progressive_summarization")
    
    # 6. Self-verification before handoff (L5, L11)
    verification = run_self_verification_checklist()
    if not verification.passed:
        # Don't hand off broken state
        raise VerificationFailed(verification.failures)
    
    # 7. Write handoff artifact (L11)
    write_handoff(task_state, trace_summary=summarize(trace))
    
    # 8. Store trace for evals (L5, L6)
    eval_store.save(trace, harness_version=harness_version)
    
    return SessionResult(success=True, trace=trace)
```

---

## Step 5: The Harness Maturity Model

Use this to assess where you are and what to improve next.

### Level 0 — No Harness
- Raw model API calls
- No persistent state, no instructions
- *Outcome*: Inconsistent, unreliable, unrepeatable

### Level 1 — Basic Instructions
- `CLAUDE.md` with codebase context
- Basic tool set
- *Outcome*: More consistent, but no systematic failure recovery

### Level 2 — Context-Managed
- Active context compression
- Handoff artifacts for multi-session tasks
- *Outcome*: Long tasks become viable

### Level 3 — Evaluated
- Eval suite with deterministic verifiers
- Trace logging with harness versioning
- Self-verification before handoff
- *Outcome*: Can measure quality, can detect regressions

### Level 4 — Controlled Autonomy
- Tool permission matrix by phase
- Sandboxed code execution
- Prompt injection defenses
- Human checkpoint triggers
- *Outcome*: Safe enough for tasks with real-world side effects

### Level 5 — Production-Grade
- All of the above
- Durable execution (crash recovery, resume)
- Multi-agent coordination with structured handoffs
- `init.sh` for reproducible environments
- Benchmark-based harness iteration
- *Outcome*: Reliable at production scale, improvable with evidence

Most teams in 2026 are at Level 1-2. Level 3 separates serious engineering from vibe-coding. Level 5 is the frontier.

---

## Final Capstone Exercise

Design a **complete production harness** for this scenario:

> **Scenario**: Your team wants to use an AI agent to handle incoming GitHub issues. For each new issue:
> 1. Triage (assign labels, priority, assign to correct team)
> 2. If it's a bug: attempt to reproduce and produce a minimal repro case
> 3. If it's a feature request: draft an initial design document
> 4. Post a first response on the issue
>
> This runs continuously, 24/7, on real production repos.

For each lecture topic, write your harness design:

| Topic | Your Design |
|-------|------------|
| **L1 — What is the harness?** | What are all the harness components? |
| **L2 — Context & Memory** | What's in context? What's summarized? What's the handoff? |
| **L3 — Constraints** | What can it do autonomously? What needs human approval? What's forbidden? |
| **L4 — Specs & Agent Files** | What's in CLAUDE.md? What spec does each issue get? |
| **L5 — Evals** | How do you know a triage was correct? What's your eval suite? |
| **L6 — Benchmarks** | Which public benchmark is closest to this task? |
| **L7 — Runtime** | What runtime infrastructure do you need? |
| **L8 — Multi-Agent** | Do you need multiple agents? What roles? |
| **L9 — Tool Design** | List 5-7 tools with signatures |
| **L10 — Advanced Context** | What backpressure + tool masking do you apply? |
| **L11 — Long-Running** | What does init.sh do? What's in the feature list? |
| **L12 — Maturity Level** | What maturity level does your design reach? What would Level 5 require? |

---

## The Closing Insight

Every lecture in this course can be summarized by one sentence:

> **The model is one component. The harness is everything else. The harness determines whether the model is useful or not.**

Better harnesses beat better models — consistently, measurably, and repeatably. The research is clear ([LangChain](https://blog.langchain.com/improving-deep-agents-with-harness-engineering/), [HumanLayer](https://www.humanlayer.dev/blog/skill-issue-harness-engineering-for-coding-agents), [Anthropic](https://www.anthropic.com/engineering/infrastructure-noise)): harness quality dominates outcome quality.

This is the central bet of harness engineering as a discipline: **the bottleneck is not the model. It's the environment around it.**

---

## Complete Course Reference

| Lecture | Topic | Core Insight |
|---------|-------|-------------|
| 1 | What is a Harness? | Model is one component; harness is everything else |
| 2 | Context & Memory | Context window = RAM; manage it deliberately |
| 3 | Constraints & Safe Autonomy | Guardrails enable speed; prompt injection needs harness defense |
| 4 | Specs & Agent Files | Persistent instructions; CLAUDE.md is the agent's onboarding doc |
| 5 | Evals & Observability | Measure harness quality, not just model quality |
| 6 | Benchmarks | Same model, better harness = better scores; control your variables |
| 7 | Runtimes | Durable execution + runtime infrastructure = production reliability |
| 8 | Multi-Agent Systems | Structured artifacts + role separation = coordination without conflict |
| 9 | Tool Design | One tool, one action; bad interfaces cause systematic agent errors |
| 10 | Advanced Context Eng. | Compress proactively, mask tools, apply backpressure |
| 11 | Long-Running Apps | init.sh + feature lists + handoff artifacts = multi-session coherence |
| 12 | Production Capstone | All pillars integrated; maturity model to assess where you are |

---

## Further Reading (Comprehensive)

- [Harness engineering: leveraging Codex in an agent-first world](https://openai.com/index/harness-engineering/) — The flagship field report
- [Effective harnesses for long-running agents](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents) — Anthropic's core article
- [Building effective agents](https://www.anthropic.com/engineering/building-effective-agents) — The broader Anthropic guide
- [12 Factor Agents](https://www.humanlayer.dev/blog/12-factor-agents) — Operating principles synthesis
- [12-Factor AgentOps](https://www.12factoragentops.com/) — Operations-oriented complement
- [Skill Issue: Harness Engineering for Coding Agents](https://www.humanlayer.dev/blog/skill-issue-harness-engineering-for-coding-agents) — Weak results = harness problems
- [Improving Deep Agents with harness engineering](https://blog.langchain.com/improving-deep-agents-with-harness-engineering/) — Measured evidence of harness impact
- [Harness Engineering (Thoughtworks)](https://martinfowler.com/articles/exploring-gen-ai/harness-engineering.html) — Context engineering, constraints, entropy
- [walkinglabs/learn-harness-engineering](https://github.com/walkinglabs/learn-harness-engineering) — Project-based course repository

---

*← [Lecture 11: Long-Running Apps](./lecture-11-long-running-apps.md)*
*→ Back to [Course Overview](./README.md)*
