# 第6讲：Benchmarks（基准测试）——衡量 Harness 质量，而不仅仅是模型质量

> **费曼核心概念**：如何用标准化基准测试客观地比较 harness，以及为什么大多数排行榜衡量的是错误的东西。

---

## 第一步：简单解释

想象两支赛车队。两队使用同一台发动机（同一模型）。但A队完成了比赛，B队在第3圈撞车了。

你可能看着比赛结果得出结论："A队的发动机更好。"但这是错的——发动机完全相同。区别在于*车身设计、轮胎、维修站策略和赛道准备*——也就是 harness。

当今大多数 AI 基准测试衡量的是错误的事情：它们衡量哪个模型 + harness 的组合胜出，然后把胜利归因于模型。这就像把每场比赛结果都归因于发动机一样。

**Harness 感知型基准测试（Harness-aware benchmarking）**用基准来回答一个不同的问题：*在使用相同模型的情况下，这个 harness 是否产生了更好的结果？*

### 类比 🏁

奥运会体育兴奋剂检测：基准测试（比赛成绩）是客观的。但要知道一名运动员的进步是来自训练（harness）还是药物（模型升级），你需要受控对比。同一运动员，不同训练方案 = harness 实验。不同运动员 = 模型实验。

---

## 第二步：识别盲区

| 盲区 | 我的说法 | 我不确定的地方 |
|------|---------|--------------|
| 1 | "基准测试衡量的是错误的东西" | 哪些基准测试实际上适合 harness 评估？ |
| 2 | "Harness 感知型基准测试" | 如何在基准测试中实际控制模型 vs harness 的变量？ |
| 3 | "SWE-bench、OSWorld 等" | 每个基准测试实际测试什么？ |
| 4 | "基础设施噪声" | 第5讲提过这个——这个效果有多大？ |

---

## 第三步：填补盲区

### 盲区1：哪些基准测试适合 Harness 评估？

**解答**：该仓库识别出了"压力测试 context 处理、工具调用、环境控制、验证逻辑，以及模型周围运行时脚手架（scaffolding）"的基准测试——即对 harness 敏感的基准测试。

最佳 harness 评估基准测试具有以下特征：
1. **基于执行的验证器（Execution-based verifiers）**——成功由运行输出来判定，而非主观评判
2. **多步任务（Multi-step tasks）**——单步任务不会暴露 harness 的优缺点
3. **真实环境（Realistic environments）**——Agent 在真实操作系统、浏览器或代码库中工作
4. **长周期（Long-horizon）**——需要多轮次的任务，会暴露漂移和记忆失败

仓库中排名靠前的对 harness 敏感的基准测试：

| 基准测试 | 测试内容 | 为什么对 harness 敏感 |
|---------|---------|-------------------|
| [SWE-bench Verified](https://www.swebench.com/) | 修复真实 GitHub issue（已有测试） | 检索、补丁应用、测试验证全部可见 |
| [OSWorld](https://os-world.github.io/) | 369个真实电脑任务（Ubuntu/Win/Mac） | 桌面 agent harness 深度 |
| [Terminal-Bench](https://www.tbench.ai/) | 终端原生 Agent（shell、文件系统） | 编码 agent harness 设计 |
| [AppWorld](https://appworld.dev/) | 应用任务中的交互式编码 Agent | 规划 + 代码生成 + 附带损害控制 |
| [AgentBench](https://github.com/THUDM/AgentBench) | 操作系统、数据库、Web、知识图谱 | Harness 在各环境中的泛化能力 |
| [WebArena](https://webarena.dev/) | 真实 Web 任务的自主 Agent | 面向 Web 的 harness 设计 |

---

### 盲区2：如何控制模型 vs Harness 的变量

**问题**：如果我的 SWE-bench 分数提高了，我怎么知道是模型还是 harness 的功劳？

**解答**（[LangChain — Improving Deep Agents with harness engineering](https://blog.langchain.com/improving-deep-agents-with-harness-engineering/)）：

LangChain 进行了一个受控实验：相同模型，不同 harness 配置。仅 harness 改变就产生了显著的基准测试改进——证明 harness 质量是独立可衡量的。

实验方案：
1. **固定模型**——实验之间不改变它
2. **每次只改变一个 harness 变量**——例如，只改变 context 压缩策略
3. **对同一基准测试版本运行**——可重现的任务集
4. **比较追踪内容**——不仅仅是分数，还有工具调用次数、token 用量、失败模式

**简单版本**：科学方法同样适用。控制变量。改变一件事。衡量结果。

---

### 盲区3：每个基准测试实际测试什么

**SWE-bench Verified** 🔧
- 任务：修复来自真实 GitHub 仓库的 bug，测试已预先编写
- 揭示的 harness 特征：代码检索策略、补丁应用、测试运行 harness
- 为什么是"Verified"：经人工验证，去除了模糊任务

**OSWorld** 💻
- 任务：跨 Ubuntu、Windows、macOS 的369个真实电脑任务
- 揭示的 harness 特征：桌面 agent harness 深度、截图/GUI 理解、多应用工作流
- 特别之处：每个任务都有初始状态设置和基于执行的评估器

**Terminal-Bench** ⌨️
- 任务：在 shell 和文件系统中工作的终端原生 Agent
- 揭示的 harness 特征：编码 agent harness 如何处理真实终端环境
- 注意：v2.0 + Harbor 本身就是一个通用评估 harness

**AppWorld** 📱
- 任务：Agent 在"一个可控制的应用和人员世界"中完成任务
- 揭示的 harness 特征：规划质量、代码生成、附带损害控制（不破坏其他事物）

**AgentBench** 🌐
- 任务：跨操作系统、数据库、知识图谱、Web 浏览的各种环境
- 揭示的 harness 特征：你的 harness 是否能泛化到单一任务循环之外

---

### 盲区4：基础设施噪声的规模

**问题**：基础设施噪声实际上对分数影响有多大？

**解答**（[Anthropic — Quantifying infrastructure noise](https://www.anthropic.com/engineering/infrastructure-noise)）：

Anthropic 发现，运行时配置更改可以使编码基准测试分数的变化幅度，超过公共排行榜上许多模型层级之间的差距。这意味着：

- 针对特定基准测试环境优化的 harness，可以在较差环境中胜过"更好"的模型
- 如果 harness 配置不同，排行榜比较通常是苹果和橙子的比较
- 你的内部基准测试环境必须与生产环境*完全*一致

**简单版本**：基础设施的"噪声底线"比大多数人想象的要高。不要相信小于基础设施噪声的排行榜差距。

---

## 第四步：精炼解释

### 如何将基准测试用于 Harness 开发

**第1步——选择符合你使用场景的对 harness 敏感的基准测试**：
- 编码任务 → SWE-bench Verified 或 Terminal-Bench
- 电脑使用 → OSWorld
- Web 任务 → WebArena 或 AssistantBench
- 多环境 → AgentBench

**第2步——建立基线**（固定模型，当前 harness）

**第3步——每次只迭代一个 harness 变量**：
```
实验1：改变 context 压缩策略 → 重新运行 → 比较
实验2：改变工具调用预算 → 重新运行 → 比较
实验3：添加自我验证步骤 → 重新运行 → 比较
```

**第4步——追踪追踪质量（trace quality）**，而不仅仅是分数：
- 任务完成率是否上升了？
- Token 成本是否下降了？
- 失败模式是否发生了变化？（新的失败通常揭示 harness 缺口）

**第5步——让 eval 环境与生产环境匹配**

### Harness 工作的基准测试层级

```
用于：开发迭代（快、便宜）
→ 单环境基准测试：Terminal-Bench、SWE-bench 子集

用于：harness 验证（中等成本）
→ 多环境：AgentBench、WebArena

用于：生产就绪（昂贵、全面）
→ 完整 OSWorld、完整 SWE-bench Verified、AppWorld
```

### 核心要点

1. 大多数基准测试分数混淆了模型质量和 harness 质量，要通过实验将它们分开。
2. 选择能暴露你实际 harness 失败模式的基准测试（长周期、工具使用、多环境）。
3. 基础设施噪声可能淹没模型差异——让 eval 环境与生产环境匹配。
4. 迭代地使用基准测试：固定模型，改变一个 harness 变量，衡量结果。

---

### 30秒电梯演讲

> "基准测试能告诉你 harness 是否在变好——但前提是你固定模型、每次只改变一个 harness 变量。大多数排行榜比较毫无意义，因为没有控制 harness 差异。"

---

## 🧪 实践 6

**基准测试分析练习**：

**A部分——选择合适的基准测试**：你正在为一个 Agent 构建 harness，该 Agent 浏览公司内部 wiki 来回答员工问题。上面列表中哪个基准测试最接近你的使用场景？为什么？

**B部分——受控实验设计**：你想测试在 harness 中添加"context 压缩步骤"是否能提高性能。写出实验方案：
- 什么保持固定？
- 什么发生变化？
- 衡量哪些指标？
- "成功"看起来是什么样的？

**C部分——基础设施噪声审计**：列出3种你的开发环境可能与生产环境不同、从而使基准测试结果失效的方式。

---

## 延伸阅读

- [SWE-bench Verified](https://www.swebench.com/) — 软件工程 Agent 基准测试
- [OSWorld](https://os-world.github.io/) — 真实电脑任务基准测试
- [AgentBench](https://github.com/THUDM/AgentBench) — 跨环境基准测试
- [Terminal-Bench](https://www.tbench.ai/) — 终端原生 Agent 基准测试
- [AppWorld](https://appworld.dev/) — 交互式编码 Agent 基准测试
- [Improving Deep Agents with harness engineering](https://blog.langchain.com/improving-deep-agents-with-harness-engineering/) — Harness 改变提高了基准测试分数
- [Quantifying infrastructure noise in agentic coding evals](https://www.anthropic.com/engineering/infrastructure-noise) — 基础设施噪声的规模

---

*← [第5讲：Evals 与可观测性](./lecture-05-evals-and-observability.md)*
*→ [第7讲：Runtimes 与参考实现](./lecture-07-runtimes-and-implementations.md)*
