# 第9讲：工具设计——构建 Agent 能正确使用的工具

> **费曼核心概念**：你的工具*设计*如何决定 Agent 是否正确使用它们——以及为什么糟糕的工具接口造成的失败多于糟糕的模型。

---

## 第一步：简单解释

想象你递给某人一把刀。如果它设计良好——平衡的手柄、清晰的刀刃方向、适合任务的尺寸——他们会正确使用它。如果是一把奇怪的刀片，手柄模糊、没有明显的"哪端切割"——他们可能会伤害自己。

AI Agent 使用工具的方式相同。设计糟糕的工具接口——模糊的参数名、不明确的输出、过多的参数、重叠的功能——会导致系统性的误用。不是因为 Agent 愚蠢，而是因为接口令人困惑。

**工具设计（Tool design）**是构建 Agent 端接口的实践，这些接口易于正确调用、难以错误调用，并产出 Agent 能理解并采取行动的输出。

### 类比 🔧

设计良好的 API vs 糟糕的 API：好的 API 有清晰的名称、最少的必需参数、可预测的错误和幂等行为。糟糕的 API 有20个重载参数、不一致的错误格式，以及从函数名看不出来的副作用。Agent 只是调用者——它们体验的是和人类使用糟糕 API 一样的痛苦。

---

## 第二步：识别盲区

| 盲区 | 我的说法 | 我不确定的地方 |
|------|---------|--------------|
| 1 | "易于正确调用、难以错误调用" | 在工具设计实践中这意味着什么？ |
| 2 | "Agent 能理解的输出" | 什么让工具输出对 Agent 来说好 vs 差？ |
| 3 | "MCP（Model Context Protocol）" | MCP 是什么，它如何融入工具设计？ |
| 4 | "代码执行工具" | 能运行代码的工具有什么特殊考虑？ |

---

## 第三步：填补盲区

### 盲区1：什么让工具易于正确调用？

**解答**（[Anthropic — Writing effective tools for agents](https://www.anthropic.com/engineering/writing-tools-for-agents)）：

Anthropic 关于 Agent 友好型工具的指南：

**名称必须是明确的动作**：
```
❌ 差：process_file(file, mode)       ← "process"是什么意思？有什么 mode？
✅ 好：read_file(path)               ← 显而易见
✅ 好：write_file(path, content)     ← 显而易见
✅ 好：append_to_file(path, content) ← 与 write_file 明显区分
```

**参数必须是最少且有类型的**：
```
❌ 差：search(query, options: {fuzzy, exact, regex, case_sensitive, max_results, offset, ...})
✅ 好：search_exact(query, max_results=10)
✅ 好：search_regex(pattern, max_results=10)
```

**每个工具只做一件事**：
```
❌ 差：manage_file(action: "read"|"write"|"delete"|"move", ...)
✅ 好：read_file(), write_file(), delete_file(), move_file()
```

**简单版本**：如果你无法用5个词解释工具做什么，就把它拆分。

---

### 盲区2：什么让工具输出对 Agent 来说好？

**解答**（Anthropic 工具设计指南）：

Agent 能良好处理的工具输出：

| 输出特性 | 为什么重要 |
|---------|-----------|
| **结构化（Structured）** | Agent 可以可靠地提取特定字段 |
| **简洁（Concise）** | 不浪费 context window 于噪声 |
| **可行动（Actionable）** | 包含 Agent 决定下一步所需的内容 |
| **错误显式（Error-explicit）** | 失败被清晰描述，而不是被静默吞噬 |

示例：
```json
// ❌ 差的输出（冗长、非结构化）
"文件读取成功。以下是您请求的位于 /src/auth.ts 的文件内容：[2000行代码]..."

// ✅ 好的输出（结构化、简洁）
{
  "status": "success",
  "path": "/src/auth.ts",
  "line_count": 2000,
  "content": "...",  // 实际内容
  "truncated": false
}

// ✅ 好的错误输出
{
  "status": "error",
  "error_type": "file_not_found",
  "path": "/src/auth.ts",
  "suggestion": "您是否指的是 /src/authentication.ts？"
}
```

---

### 盲区3：Model Context Protocol（MCP）

**问题**：MCP 是什么，为什么对工具设计重要？

**解答**（[Anthropic — Code execution with MCP](https://www.anthropic.com/engineering/code-execution-with-mcp)）：

MCP（Model Context Protocol，模型上下文协议）是 AI 模型如何连接到外部工具和数据源的标准。不是每个 harness 都自己发明工具调用接口，MCP 提供了一个共享协议。

主要优势：
- **可检查性（Inspectability）**——每次工具调用通过协议显式记录
- **可组合性（Composability）**——为 MCP 构建的工具可以与任何兼容 MCP 的模型/harness 一起工作
- **安全性（Security）**——工具权限和边界是协议的一等概念

从 harness 设计角度：MCP 给了你"执行边界"。当 Agent 通过 MCP 调用工具时，harness 可以：
1. 在执行前记录调用
2. 根据 schema 验证参数
3. 检查授权（这个 Agent 可以调用这个工具吗？）
4. 在沙盒环境中执行
5. 记录结果

**简单版本**：MCP 是工具的 USB 标准——不是每个工具都需要自定义驱动，它们都使用同一个协议，harness 处理其余的事情。

专门测试 MCP 质量的基准测试：
- [MCP Bench](https://github.com/modelscope/MCPBench) — 工具精准度、延迟、token 用量
- [MCPMark](https://github.com/eval-sys/mcpmark) — 跨 Notion、GitHub、Postgres 的压力测试
- [OSWorld-MCP](https://osworld-mcp.github.io) — 通过 MCP 的真实世界电脑任务

---

### 盲区4：代码执行工具

**问题**：能实际运行代码的工具有什么特殊考虑？

**解答**（[Anthropic — Code execution with MCP](https://www.anthropic.com/engineering/code-execution-with-mcp) + [SWE-ReX](https://github.com/SWE-agent/SWE-ReX)）：

代码执行是最高风险的工具类别。Harness 必须：

1. **沙盒化一切（Sandbox everything）**——代码在隔离环境中运行，而不是在主机系统上
2. **资源限制（Resource limits）**——CPU 时间、内存、网络访问被显式限制
3. **输出捕获（Output capture）**——stdout、stderr、退出码都返回给 Agent
4. **超时处理（Timeout handling）**——挂起的进程被终止并报告，而不是静默等待
5. **状态隔离（State isolation）**——每次执行从已知状态开始（或显式继承状态）

```
Agent 调用：run_python("import os; os.system('rm -rf /')")
Harness：
  1. 接收工具调用
  2. 路由到沙盒（不是主机操作系统）
  3. 沙盒以有限权限运行
  4. 返回：{exit_code: 1, stderr: "Permission denied"}
  5. 日志：{tool: "run_python", input: "...", result: "...", sandbox_id: "..."}
```

**简单版本**：永远不要在你的真实系统上运行 Agent 生成的代码。始终沙盒化。始终捕获输出。始终记录日志。

---

## 第四步：精炼解释

### 工具设计检查清单

对于 harness 中的每个工具，验证：

- [ ] **名称是明确的动作动词**（read、write、search、delete——不是"manage"或"process"）
- [ ] **只做一件事**（拆分多动作工具）
- [ ] **最少的必需参数**（没有特殊原因，必需参数不超过5个）
- [ ] **所有参数命名清晰**（没有不带具体说明的 `mode`、`type`、`options`）
- [ ] **输出是结构化且简洁的**（JSON > 散文，过滤后的 > 原始的）
- [ ] **错误是显式且可行动的**（什么失败了 + 可以尝试什么）
- [ ] **尽可能幂等（Idempotent）**（调用两次不会产生双倍的副作用）
- [ ] **危险操作需要显式确认**（或在 harness 层面处理）

### 工具表面积原则（Tool Surface Area Principle）

少即是多。你给 Agent 的每个工具：
- 增加了 Agent 必须推理的决策空间
- 创造了更多被误用的表面积
- 使 harness 更难审计和测试

从能完成任务的最小工具集开始。只有在有具体的未满足需求时才添加工具，而不是"以防万一"。

### 核心要点

1. 工具接口质量直接决定 Agent 行为质量——糟糕的接口导致系统性错误。
2. 一个工具，一个动作。清晰的名称、最少的参数、结构化的输出。
3. MCP 是 Agent 工具的标准协议——使用它来获得可检查性、可组合性和安全性。
4. 代码执行工具需要明确的沙盒化——无例外。

---

### 30秒电梯演讲

> "工具是 Agent 的双手。如果手柄令人困惑，双手就会做错事。清晰的名称、每个工具一个动作、结构化的输出，以及代码执行的强制沙盒化。"

---

## 🧪 实践 9

**工具设计练习**：

**A部分——批评这些工具**：对于下面每个糟糕的工具，解释问题所在并重新设计它：

```python
# 工具1
def file_manager(action: str, path: str, content: str = None, 
                  dest: str = None, mode: str = "r"):
    ...

# 工具2  
def search(query: str, fuzzy: bool = False, regex: bool = False,
           case_sensitive: bool = True, max_results: int = 100,
           include_tests: bool = True, file_types: list = None):
    ...

# 工具3
def database(operation: str, table: str, data: dict = None, 
             where: str = None, join: str = None):
    ...
```

**B部分——设计一套工具**：你正在为一个代码审查 Agent 构建 harness。它需要：读取文件、搜索模式、留下评论、运行测试。设计最小工具集（最多5个工具）。对于每个工具：
- 名称
- 参数（带类型）
- 返回类型/格式
- 它**不**做什么

**C部分——输出质量**：下面的工具返回原始日志转储。重新设计输出使其对 Agent 友好：

```
工具：run_tests()
当前输出："Running test suite...\n[2026-04-01 10:23:11] Starting test runner\n
[2026-04-01 10:23:11] Loading configuration from jest.config.js\n
[2026-04-01 10:23:12] Found 47 test suites\n[2026-04-01 10:23:12] Running...\n
PASS src/auth.test.ts (2.3s)\nFAIL src/orders.test.ts\n  ● OrderService › createOrder › 
should validate items\n    Expected: [{id: 1, qty: 2}]\n    Received: undefined\n
PASS src/users.test.ts (1.1s)\n[2026-04-01 10:23:18] Test Suites: 1 failed, 46 passed\n
Tests: 1 failed, 203 passed\nTime: 6.2s"
```

它应该返回什么？

---

## 延伸阅读

- [Writing effective tools for agents](https://www.anthropic.com/engineering/writing-tools-for-agents) — Anthropic 关于 Agent 友好型工具接口的指南
- [Code execution with MCP](https://www.anthropic.com/engineering/code-execution-with-mcp) — 通过显式工具边界实现受控执行能力
- [SWE-ReX](https://github.com/SWE-agent/SWE-ReX) — 沙盒执行基础设施
- [MCP Bench](https://github.com/modelscope/MCPBench) — 基准测试 MCP 工具质量
- [MCPMark](https://github.com/eval-sys/mcpmark) — 压力测试真实世界 MCP 任务

---

*← [第8讲：多 Agent 系统](./lecture-08-multi-agent-systems.md)*
*→ [第10讲：Context Engineering 进阶](./lecture-10-context-engineering-advanced.md)*
