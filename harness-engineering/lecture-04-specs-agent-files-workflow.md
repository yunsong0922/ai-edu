# Lecture 4: Specs, Agent Files & Workflow Design

> **Feynman Concept**: How to give agents durable, structured instructions that survive across sessions and scale across teams.

---

## Step 1: Explain It Simply

You hire a contractor to renovate your kitchen. You don't follow them around all day explaining what you want. You hand them a **spec sheet**: the dimensions, the materials, the style, what NOT to touch.

When agents work on a codebase, they need the same thing — a persistent document they can always refer back to. Without it, every session starts from scratch. The agent doesn't know your conventions, your sensitive files, or your preferred testing approach.

**Agent files** (`CLAUDE.md`, `AGENTS.md`, `agent.md`) are that spec sheet. They live in the repo, persist across sessions, and tell the agent "this is how we do things here."

### Analogy 📋

Employee handbook + job description: The job description tells you *what* to do. The handbook tells you *how we do things here* — the culture, the rules, the things that got someone fired last year. Agent files are both.

---

## Step 2: Identify Gaps

| Gap | What I Said | What I'm Not Sure About |
|-----|-------------|------------------------|
| 1 | "Persistent across sessions" | How do agent files actually get loaded? Who reads them? |
| 2 | "CLAUDE.md vs AGENTS.md vs agent.md" | What's the actual difference between these formats? |
| 3 | "12-Factor Agents" | What are these principles and why do they matter? |
| 4 | "Spec-driven development" | How do specs change the agent workflow vs ad-hoc prompts? |

---

## Step 3: Fill the Gaps

### Gap 1: How Agent Files Get Loaded

**The question**: When does the agent actually read `CLAUDE.md`? Is it automatic?

**The answer** ([HumanLayer — Writing a good CLAUDE.md](https://www.humanlayer.dev/blog/writing-a-good-claude-md)):

Claude Code automatically reads `CLAUDE.md` at the start of every session (if present in the working directory or any parent directory). The harness can also instruct the agent to read domain-specific files at task start — e.g., `"before touching any module, read ARCHITECTURE.md"`.

The file must be:
1. **Present in the repo** — not external, not ephemeral
2. **In plain Markdown** — the agent reads it like a document
3. **Trusted by the harness** — not just any file, but one the harness explicitly loads

**Simple version**: It's like a `.bashrc` for your agent — automatically sourced on every session start.

---

### Gap 2: CLAUDE.md vs AGENTS.md vs agent.md

**The question**: These three formats exist. What's actually different between them?

**The answer**:

| Format | Source | Scope | Key difference |
|--------|--------|-------|----------------|
| `CLAUDE.md` | Anthropic | Claude Code-specific | Tool-specific, rich support in CC |
| `AGENTS.md` | [agentsmd/agents.md](https://github.com/agentsmd/agents.md) | Open standard, tool-agnostic | Works across Claude, Codex, Cursor, etc. |
| `agent.md` | [agentmd/agent.md](https://github.com/agentmd/agent.md) | Related open standard | Slightly different schema, same goal |

In practice: pick one, put it in your repo root, and be consistent. The content matters more than the filename.

**Simple version**: Different dialects of the same language — "here's how to work in this codebase."

---

### Gap 3: 12-Factor Agents

**The question**: What are the 12-Factor Agents principles?

**The answer** ([HumanLayer — 12-Factor Agents](https://www.humanlayer.dev/blog/12-factor-agents)):

Modeled on the Twelve-Factor App manifesto for web services. Key principles for agent harness design:

| Factor | Principle | What it means |
|--------|-----------|--------------|
| Explicit prompts | No hidden context | Every instruction the agent receives is visible and auditable |
| Stateless agents | Harness owns state | The agent doesn't accumulate state between runs — the harness does |
| Declarative tools | Treat tools like functions | Tools are stateless, side-effect-free where possible |
| Pause-resume | Clean interruption | The agent can be stopped and resumed from a known checkpoint |
| Separation of concerns | Logic vs plumbing | Agent logic separate from runtime infrastructure (retries, logging) |
| Observable | Traces everything | Every agent action is logged and inspectable |

**Simple version**: Just like good microservices — stateless, explicit, resumable, and observable.

---

### Gap 4: Spec-Driven Development

**The question**: How does writing a spec before the agent starts change the outcome?

**The answer** ([Thoughtworks — Understanding Spec-Driven-Development](https://martinfowler.com/articles/exploring-gen-ai/sdd-3-tools.html) + [GitHub Spec Kit](https://github.com/github/spec-kit)):

Without a spec, the agent interprets the goal ambiguously at every turn. Small misinterpretations accumulate. You end up reviewing hundreds of lines of "almost right" code.

With a spec, the agent has an explicit reference to check its work against. Each decision point becomes: "does this satisfy the spec?" instead of "does this feel right?"

[12-Factor AgentOps](https://www.12factoragentops.com/) adds an operations lens: specs also make agent workflows **reproducible** — if you can run the same spec twice and get the same outcome, you can debug, improve, and A/B test your harness.

**Simple version**: Specs are like unit tests for the agent's understanding — they make "correct" concrete and measurable.

---

## Step 4: Refined Explanation

### What Makes a Good Agent File

```markdown
# CLAUDE.md — Example Structure

## What This Codebase Does
A REST API for e-commerce order management.
Uses Node.js + Express + PostgreSQL.

## Architecture
- /src/routes/        — HTTP route handlers (thin, delegate to services)
- /src/services/      — Business logic (no direct DB access here)
- /src/repositories/  — Database layer (Postgres via pg library)
- /src/models/        — Type definitions (TypeScript interfaces)

## Conventions
- Always use the Repository pattern for DB access (never query in routes)
- All async functions must have explicit error handling (no silent failures)
- New endpoints require an integration test in /tests/integration/

## DO NOT Touch
- /src/legacy/        — Deprecated, migration in progress (ask before any change)
- .env files          — Never read, modify, or log these

## Testing
Run before marking any task done:
  npm test              (unit tests — must all pass)
  npm run test:int      (integration tests — must all pass)

## Handoff
When completing a task, create TASK_STATUS.md with:
- What was done
- What tests were run
- What remains incomplete
- Any decisions made and why
```

### The Agent File Maturity Ladder

| Level | What Exists | Agent Behavior |
|-------|-------------|----------------|
| 0 — None | No agent file | Agent guesses conventions, makes inconsistent choices |
| 1 — Basic | Lists what the codebase does | Fewer wrong-codebase assumptions |
| 2 — Conventions | Adds do/don't rules | Consistent style and patterns |
| 3 — Boundaries | Adds "don't touch" list | Avoids sensitive areas |
| 4 — Verification | Adds test commands | Self-verifies before declaring done |
| 5 — Handoff | Adds artifact spec | Sessions resume cleanly |

Aim for Level 4+.

### Key Takeaways

1. Agent files are the harness's persistent memory — they survive across every session.
2. Good agent files specify what NOT to do, not just what to do.
3. Specs enable deterministic, reproducible agent workflows — no ambiguity, no accumulating misinterpretation.
4. Treat agent files as living documents — update them when the agent makes a surprising mistake.

---

### 30-Second Elevator Pitch

> "Agent files are persistent instructions in your repo. Without them, every session starts from scratch and conventions drift. With them, the agent knows your standards from line one."

---

## 🧪 Practice 4

**Writing exercise**: Write a `CLAUDE.md` for a hypothetical project — a REST API for a small e-commerce store.

Include ALL of these sections:

1. **What the codebase does** — 2 sentences max
2. **Architecture** — list the key folders/modules and what each does
3. **Conventions** — 2 rules the agent MUST follow (e.g., testing patterns, error handling)
4. **DO NOT Touch** — 1 area that is off-limits without human approval
5. **Testing** — the exact command(s) to run before marking a task done
6. **Handoff** — what the agent must leave behind when it finishes

**Bonus**: Identify one thing that a naive agent would get wrong in your codebase that your `CLAUDE.md` prevents.

---

## Further Reading

- [AGENTS.md open format](https://github.com/agentsmd/agents.md) — Machine-readable agent instructions across tools
- [Writing a good CLAUDE.md](https://www.humanlayer.dev/blog/writing-a-good-claude-md) — Practical guide to durable repo-local instructions
- [12 Factor Agents](https://www.humanlayer.dev/blog/12-factor-agents) — Operating principles for production agents
- [12-Factor AgentOps](https://www.12factoragentops.com/) — Context discipline, validation, reproducible workflows
- [GitHub Spec Kit](https://github.com/github/spec-kit) — Toolkit for spec-driven development
- [Understanding Spec-Driven-Development](https://martinfowler.com/articles/exploring-gen-ai/sdd-3-tools.html) — Thoughtworks on why strong specs make AI delivery more dependable
- [Claude Code: Best practices for agentic coding](https://code.claude.com/docs) — Anthropic's practical recommendations

---

*← [Lecture 3: Constraints & Safe Autonomy](./lecture-03-constraints-and-safe-autonomy.md)*
*→ [Lecture 5: Evals & Observability](./lecture-05-evals-and-observability.md)*
