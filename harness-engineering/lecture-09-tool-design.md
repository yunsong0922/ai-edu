# Lecture 9: Tool Design — Building Tools Agents Can Use Correctly

> **Feynman Concept**: How the *design* of your tools determines whether agents use them correctly — and why bad tool interfaces cause more failures than bad models.

---

## Step 1: Explain It Simply

Imagine you hand someone a knife. If it's a well-designed knife — balanced handle, clear blade direction, appropriate size for the task — they use it correctly. If it's a weird blade with an ambiguous handle and no obvious "which end cuts" — they might hurt themselves.

AI agents use tools the same way. A poorly designed tool interface — vague parameter names, ambiguous outputs, too many parameters, overlapping functionality — leads to systematic misuse. Not because the agent is dumb, but because the interface is confusing.

**Tool design** is the practice of building agent-facing interfaces that are easy to call correctly, hard to call incorrectly, and produce outputs the agent can understand and act on.

### Analogy 🔧

A well-designed API vs. a bad one: A good API has clear names, minimal required parameters, predictable errors, and idempotent behavior. A bad API has 20 overloaded parameters, inconsistent error formats, and side effects that aren't obvious from the function name. Agents are just callers — they experience the same pain humans do with bad APIs.

---

## Step 2: Identify Gaps

| Gap | What I Said | What I'm Not Sure About |
|-----|-------------|------------------------|
| 1 | "Easy to call correctly, hard to call incorrectly" | What does this mean in practice for tool design? |
| 2 | "Outputs the agent can understand" | What makes a tool output good vs. bad for an agent? |
| 3 | "MCP" | What is the Model Context Protocol and how does it fit into tool design? |
| 4 | "Code execution tools" | Special considerations for tools that run code? |

---

## Step 3: Fill the Gaps

### Gap 1: What Makes a Tool Easy to Call Correctly?

**The answer** ([Anthropic — Writing effective tools for agents](https://www.anthropic.com/engineering/writing-tools-for-agents)):

Anthropic's guidelines for agent-friendly tools:

**Names must be unambiguous actions**:
```
❌ Bad:  process_file(file, mode)       ← what does "process" mean? what modes?
✅ Good: read_file(path)               ← obvious
✅ Good: write_file(path, content)     ← obvious
✅ Good: append_to_file(path, content) ← distinct from write_file
```

**Parameters must be minimal and typed**:
```
❌ Bad:  search(query, options: {fuzzy, exact, regex, case_sensitive, max_results, offset, ...})
✅ Good: search_exact(query, max_results=10)
✅ Good: search_regex(pattern, max_results=10)
```

**Each tool does exactly one thing**:
```
❌ Bad:  manage_file(action: "read"|"write"|"delete"|"move", ...)
✅ Good: read_file(), write_file(), delete_file(), move_file()
```

**Simple version**: If you can't explain what the tool does in 5 words, split it up.

---

### Gap 2: What Makes a Tool Output Good for Agents?

**The answer** (Anthropic tool design guidelines):

Tool outputs that agents handle well:

| Output property | Why it matters |
|----------------|----------------|
| **Structured** | Agent can extract specific fields reliably |
| **Concise** | Doesn't waste context window on noise |
| **Actionable** | Contains what the agent needs to decide next step |
| **Error-explicit** | Failures described clearly, not swallowed silently |

Example:
```json
// ❌ Bad output (verbose, unstructured)
"The file was successfully read. Here are the contents of the file located at 
/src/auth.ts which you requested: [2000 lines of code]..."

// ✅ Good output (structured, concise)
{
  "status": "success",
  "path": "/src/auth.ts",
  "line_count": 2000,
  "content": "...",  // actual content
  "truncated": false
}

// ✅ Good error output
{
  "status": "error",
  "error_type": "file_not_found",
  "path": "/src/auth.ts",
  "suggestion": "Did you mean /src/authentication.ts?"
}
```

---

### Gap 3: The Model Context Protocol (MCP)

**The question**: What is MCP and why does it matter for tool design?

**The answer** ([Anthropic — Code execution with MCP](https://www.anthropic.com/engineering/code-execution-with-mcp)):

MCP (Model Context Protocol) is a standard for how AI models connect to external tools and data sources. Instead of each harness inventing its own tool-calling interface, MCP provides a shared protocol.

Key benefits:
- **Inspectability** — every tool call is explicit and logged via the protocol
- **Composability** — tools built for MCP work with any MCP-compatible model/harness
- **Security** — tool permissions and boundaries are first-class protocol concepts

From a harness design perspective: MCP gives you the "execution boundary." When the agent calls a tool via MCP, the harness can:
1. Log the call before it executes
2. Validate parameters against the schema
3. Check authorization (can this agent call this tool?)
4. Execute in a sandboxed environment
5. Log the result

**Simple version**: MCP is a USB standard for tools — instead of every tool needing a custom driver, they all speak the same protocol and the harness handles the rest.

Benchmarks specifically testing MCP quality:
- [MCP Bench](https://github.com/modelscope/MCPBench) — tool accuracy, latency, token use
- [MCPMark](https://github.com/eval-sys/mcpmark) — stress tests across Notion, GitHub, Postgres
- [OSWorld-MCP](https://osworld-mcp.github.io) — real-world computer tasks via MCP

---

### Gap 4: Code Execution Tools

**The question**: Special considerations for tools that actually run code?

**The answer** ([Anthropic — Code execution with MCP](https://www.anthropic.com/engineering/code-execution-with-mcp) + [SWE-ReX](https://github.com/SWE-agent/SWE-ReX)):

Code execution is the highest-risk tool category. The harness must:

1. **Sandbox everything** — code runs in an isolated environment, not the host system
2. **Resource limits** — CPU time, memory, network access explicitly bounded
3. **Output capture** — stdout, stderr, exit code all returned to agent
4. **Timeout handling** — hanging processes killed and reported, not just silence
5. **State isolation** — each execution starts in a known state (or explicitly inherits state)

```
Agent calls: run_python("import os; os.system('rm -rf /')")
Harness:
  1. Receives tool call
  2. Routes to sandbox (not host OS)
  3. Sandbox runs with limited permissions
  4. Returns: {exit_code: 1, stderr: "Permission denied"}
  5. Logs: {tool: "run_python", input: "...", result: "...", sandbox_id: "..."}
```

**Simple version**: Never run agent-generated code on your real system. Always sandbox. Always capture output. Always log.

---

## Step 4: Refined Explanation

### The Tool Design Checklist

For every tool in your harness, verify:

- [ ] **Name is an unambiguous action verb** (read, write, search, delete — not "manage" or "process")
- [ ] **Does exactly one thing** (split multi-action tools)
- [ ] **Minimal required parameters** (no >5 required params without good reason)
- [ ] **All parameters named clearly** (no `mode`, `type`, `options` without specifics)
- [ ] **Output is structured and concise** (JSON > prose, filtered > raw)
- [ ] **Errors are explicit and actionable** (what failed + what to try instead)
- [ ] **Idempotent where possible** (calling it twice doesn't double the side effect)
- [ ] **Dangerous operations require explicit confirmation** (or are handled at harness level)

### Tool Surface Area Principle

Less is more. Every tool you give an agent:
- Increases the decision space the agent must reason over
- Creates more surface area for misuse
- Makes the harness harder to audit and test

Start with the minimum tool set that accomplishes the task. Add tools only when you have a concrete unmet need, not "just in case."

### Key Takeaways

1. Tool interface quality directly determines agent behavior quality — bad interfaces cause systematic errors.
2. One tool, one action. Clear names, minimal parameters, structured outputs.
3. MCP is the standard protocol for agent tools — use it to get inspectability, composability, and security.
4. Code execution tools require explicit sandboxing — no exceptions.

---

### 30-Second Elevator Pitch

> "Tools are the agent's hands. If the handles are confusing, the hands do the wrong thing. Clear names, one action per tool, structured outputs, and mandatory sandboxing for code execution."

---

## 🧪 Practice 9

**Tool design exercise**:

**Part A — Critique these tools**: For each bad tool below, explain what's wrong and redesign it:

```python
# Tool 1
def file_manager(action: str, path: str, content: str = None, 
                  dest: str = None, mode: str = "r"):
    ...

# Tool 2  
def search(query: str, fuzzy: bool = False, regex: bool = False,
           case_sensitive: bool = True, max_results: int = 100,
           include_tests: bool = True, file_types: list = None):
    ...

# Tool 3
def database(operation: str, table: str, data: dict = None, 
             where: str = None, join: str = None):
    ...
```

**Part B — Design a tool set**: You're building a harness for a code review agent. It needs to: read files, search for patterns, leave comments, and run tests. Design the minimal tool set (max 5 tools). For each tool:
- Name
- Parameters (with types)
- Return type/format
- What it does NOT do

**Part C — Output quality**: The following tool returns a raw log dump. Redesign the output to be agent-friendly:

```
Tool: run_tests()
Current output: "Running test suite...\n[2026-04-01 10:23:11] Starting test runner\n
[2026-04-01 10:23:11] Loading configuration from jest.config.js\n
[2026-04-01 10:23:12] Found 47 test suites\n[2026-04-01 10:23:12] Running...\n
PASS src/auth.test.ts (2.3s)\nFAIL src/orders.test.ts\n  ● OrderService › createOrder › 
should validate items\n    Expected: [{id: 1, qty: 2}]\n    Received: undefined\n
PASS src/users.test.ts (1.1s)\n[2026-04-01 10:23:18] Test Suites: 1 failed, 46 passed\n
Tests: 1 failed, 203 passed\nTime: 6.2s"
```

What should this return instead?

---

## Further Reading

- [Writing effective tools for agents](https://www.anthropic.com/engineering/writing-tools-for-agents) — Anthropic's guidance on agent-friendly tool interfaces
- [Code execution with MCP](https://www.anthropic.com/engineering/code-execution-with-mcp) — Controlled execution power via explicit tool boundaries
- [SWE-ReX](https://github.com/SWE-agent/SWE-ReX) — Sandboxed execution infrastructure
- [MCP Bench](https://github.com/modelscope/MCPBench) — Benchmarking MCP tool quality
- [MCPMark](https://github.com/eval-sys/mcpmark) — Stress-testing real-world MCP tasks

---

*← [Lecture 8: Multi-Agent Systems](./lecture-08-multi-agent-systems.md)*
*→ [Lecture 10: Context Engineering Advanced](./lecture-10-context-engineering-advanced.md)*
