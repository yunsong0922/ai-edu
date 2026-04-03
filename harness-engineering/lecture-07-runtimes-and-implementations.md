# 第7讲：Runtimes 与参考实现

> **费曼核心概念**：Runtime 到底是什么，它与 harness 有何不同，以及通过阅读真实开源 harness 实现你能学到什么。

---

## 第一步：简单解释

你已经在纸上设计好了 harness——指令、约束、evals。现在你需要实际*运行*它。这就是 **runtime（运行时）**的用武之地。

这样来理解：食谱是 harness（指令、约束、步骤）。厨房是 runtime（食谱执行的环境——烤箱温度、锅具、计时器）。一个糟糕的厨房可以毁掉一份完美的食谱。

Runtimes 处理模型调用*之间*发生的事情：
- **状态（State）**——任务目前进展到哪了？什么已经完成了？
- **重试（Retries）**——如果一次工具调用失败了，会发生什么？
- **并发（Concurrency）**——多个 Agent 可以并行运行吗？
- **持久性（Durability）**——如果进程在20步中的第7步崩溃了，能恢复吗？
- **追踪（Traces）**——所有内容的日志去哪了？

### 类比 🏗️

建筑工程项目：蓝图（harness）告诉工人建什么。项目管理系统（runtime）追踪谁在做什么、处理延误、在有人生病时重新分配任务，并在一个分包商迟到时防止整个项目崩溃。

---

## 第二步：识别盲区

| 盲区 | 我的说法 | 我不确定的地方 |
|------|---------|--------------|
| 1 | "Runtime 处理状态、重试、并发" | 在实际代码中这是什么样子的？ |
| 2 | "参考实现" | 读 SWE-agent 我能实际学到什么？ |
| 3 | "Framework vs runtime vs harness" | 用具体例子重新审视这个区别 |
| 4 | "持久化执行（Durable execution）" | "持久化"对 Agent 意味着什么，为什么重要？ |

---

## 第三步：填补盲区

### 盲区1：Runtime 功能在代码中的样子

**解答**（[Inngest — Your Agent Needs a Harness, Not a Framework](https://www.inngest.com/blog/your-agent-needs-a-harness-not-a-framework)）：

Inngest 的论点：状态、重试、追踪和并发是*基础设施*——它们不属于你 Agent 的 prompt 或工具逻辑。它们属于 runtime 层。

具体来说：

```
没有 runtime 基础设施：
  agent.run(task) → 如果在第7步崩溃，从第1步重新开始

有 runtime 基础设施（例如 Inngest/AgentKit）：
  agent.run(task) → 每步后持久化状态
                  → 如果在第7步崩溃，从第7步恢复
                  → 工具调用失败时指数退避重试
                  → 无论结果如何，完整追踪都被存储
```

**简单版本**：Runtime 基础设施让你的 harness 能够*在故障中存活*。没有它，任何失败都意味着从头开始。

---

### 盲区2：从 SWE-agent 中学到什么

**解答**（[SWE-agent GitHub](https://github.com/SWE-agent/SWE-agent)）：

SWE-agent 是现有最具可检查性的 harness 实现之一。阅读它能教会你：

1. **ACI（Agent-Computer Interface，Agent-计算机接口）设计**——工具如何呈现给 Agent 至关重要。SWE-agent 的工具接口经过精心设计，简洁、可组合且难以误用。

2. **Agent 循环**——驱动 Agent 的实际 while 循环：观察 → 思考 → 行动 → 观察 → ...

3. **环境设置**——Agent 开始工作前基准测试环境如何初始化（干净的状态，正确的依赖项）

4. **大规模 Prompt 工程**——系统 prompts、任务描述和工具文档如何为可靠性而构建

5. **轨迹日志记录**——每个动作以结构化格式存储，用于 eval 和调试

**简单版本**：SWE-agent 是一本你可以运行的教科书。它展示了"我有一个 harness 的想法"和"我有一个生产级 harness 实现"之间的差距。

---

### 盲区3：Framework vs Runtime vs Harness——具体例子

| 层级 | 示例 | 提供什么 |
|------|------|---------|
| **Framework（框架）** | LangChain、LlamaIndex、CrewAI | 构建 Agent 的 API——chains、工具、记忆抽象 |
| **Runtime（运行时）** | Inngest、AgentKit、SWE-ReX | 执行基础设施——状态持久化、重试、持久性、追踪 |
| **Harness（驾具）** | 你的 CLAUDE.md + 工具配置 + eval 套件 | Agent *做什么*及如何约束它的*设计* |

你可以不用 framework 来构建 harness（仅用 prompts + 工具）。你可以使用 framework 而没有 runtime（在第一次崩溃之前都能工作）。Runtime 是让 harness 达到生产级的东西。

---

### 盲区4：持久化执行（Durable Execution）

**问题**：对 Agent 而言，"持久化"意味着什么？

**解答**（[AgentKit — Inngest](https://github.com/inngest/agent-kit)）：

持久化执行意味着系统保证每一步要么完成要么被重试——即使进程崩溃、网络故障或工具超时。对于长时间运行的 Agent：

- 一个30分钟的编码任务可能有50多次工具调用
- 没有持久性：一次网络超时 = 30分钟工作损失
- 有持久性：一次网络超时 = 重试那一次工具调用，从中断处继续

**简单版本**：持久化 = Agent 可以在任何时刻被中断并从中断处精确恢复，不丢失任何工作。

---

## 第四步：精炼解释

### Runtime 技术栈

```
你的 Agent
    ↓
Harness 层         ← 你的 CLAUDE.md、工具配置、约束、evals
    ↓
Framework 层       ← LangChain / 原始 SDK / 自定义循环
    ↓
Runtime 层         ← 状态持久化、重试、追踪、并发
    ↓
基础设施            ← 计算资源、沙盒、网络、存储
```

每一层都有自己的职责。不要让 harness 做 runtime 的工作，反之亦然。

### 值得学习的关键参考实现

**[SWE-agent](https://github.com/SWE-agent/SWE-agent)** — 从这里开始。
- 最有文档记录的开源编码 agent harness
- 学习：ACI 设计、agent 循环、轨迹日志记录、环境设置
- 在本地针对一个简单任务运行它并阅读其追踪内容

**[SWE-ReX](https://github.com/SWE-agent/SWE-ReX)**
- 从 SWE-agent 中提取的沙盒执行基础设施
- 学习：如何在不给予系统访问权限的情况下赋予 Agent 安全的执行能力

**[AgentKit by Inngest](https://github.com/inngest/agent-kit)**
- 用于持久化、工作流感知型 Agent 的 TypeScript 工具集
- 学习：在事件驱动基础设施中持久化执行在实践中如何工作

**[deepagents by LangChain](https://github.com/langchain-ai/deepagents)**
- 具有中间件和 harness 模式的长时间运行 Agent
- 学习：中间件层在真实 Agent 系统中如何工作

**[Harbor](https://github.com/harbor-framework/harbor)**
- 用于大规模评估和改进 Agent 的通用 harness
- 学习：如何构建跨多种 Agent 配置工作的 eval 基础设施

### 核心要点

1. Runtime ≠ Harness。Runtime 处理基础设施关注点（状态、重试、追踪）。Harness 处理设计关注点（做什么、约束、验证）。
2. 读 SWE-agent。它是现有最具指导意义的开源 harness 实现。
3. 持久化执行对于生产级长时间运行 Agent 不是可选项。
4. "在演示中有效"和"可靠地有效"之间的差距，几乎完全在 runtime 层。

---

### 30秒电梯演讲

> "Runtime 是你的食谱运行的厨房。没有持久化状态管理、重试逻辑和完整追踪，再好的 harness 设计也会失败。读 SWE-agent，看看生产级实现实际上是什么样子。"

---

## 🧪 实践 7

**实现练习**：

**A部分——层级分离**：你正在构建一个编码 Agent。对于下面每一项，判断它属于 *harness*、*runtime* 还是 *framework* 层：
- "不要修改 /src/legacy/ 中的文件"
- "如果工具调用因超时失败，重试最多3次"
- "记录每次工具调用及其参数和结果"
- "Agent 在宣告完成前应该始终运行测试"
- "如果进程崩溃，从最后一个成功步骤恢复"
- "所有数据库访问使用 Repository 模式"

**B部分——SWE-agent 学习**：克隆 SWE-agent 并在一个简单任务上运行它（或阅读其源码）。回答：
1. "Agent 循环"是什么——驱动 Agent 的核心 while 循环？
2. 它如何记录轨迹（trajectories）？使用什么格式？
3. 当工具调用返回错误时会发生什么？

**C部分——持久性设计**：你的 Agent 正在运行一个20步的重构任务。在第12步时，服务器崩溃了。设计一个检查点/恢复策略：
- 状态何时被持久化？
- 持久化什么状态？（提示：从第12步恢复需要什么？）
- Agent 如何"知道"自己是在恢复还是重新开始？

---

## 延伸阅读

- [Agent Frameworks, Runtimes, and Harnesses, Oh My!](https://blog.langchain.com/agent-frameworks-runtimes-and-harnesses-oh-my/) — LangChain 的分解
- [Your Agent Needs a Harness, Not a Framework](https://www.inngest.com/blog/your-agent-needs-a-harness-not-a-framework) — 状态、重试、追踪作为一等基础设施
- [SWE-agent](https://github.com/SWE-agent/SWE-agent) — 最具指导意义的生产级编码 harness
- [SWE-ReX](https://github.com/SWE-agent/SWE-ReX) — 沙盒执行基础设施
- [AgentKit](https://github.com/inngest/agent-kit) — 持久化、工作流感知型 Agent 工具集
- [deepagents](https://github.com/langchain-ai/deepagents) — 具有 harness 模式的长时间运行 Agent
- [Harbor](https://github.com/harbor-framework/harbor) — 大规模 eval 的通用 harness
- [Building agents with the Claude Agent SDK](https://claude.com/blog/building-agents-with-the-claude-agent-sdk) — Anthropic 面向生产的 SDK

---

*← [第6讲：Benchmarks](./lecture-06-benchmarks.md)*
*→ [第8讲：多 Agent 系统](./lecture-08-multi-agent-systems.md)*
