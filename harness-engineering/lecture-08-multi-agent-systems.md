# 第8讲：多 Agent 系统

> **费曼核心概念**：如何为多个 Agent 协同工作的系统设计 harness——以及为什么协调比看起来要难得多。

---

## 第一步：简单解释

一个 Agent 就已经够难了。现在想象10个 Agent 同时在同一个代码库上工作。

Agent A 正在重构认证模块。Agent B 正在为它编写测试。Agent C 正在更新 API 文档。它们都在同时工作，而且对代码应该是什么样子都有自己的"意见"。

没有精心的 harness 设计，你会得到：
- Agent A 和 B 写出了同一文件的冲突版本
- Agent C 记录了一个 Agent A 刚刚重命名的 API
- Agent B 的测试失败了，因为 Agent A 的重构还没完成
- 没有人知道整个任务的"完成"意味着什么

**多 Agent harness 设计**是给每个 Agent 明确定义的角色、清晰的边界和协调机制的实践——让它们相互放大而不是相互争斗。

### 类比 🎭

电影剧组：导演（编排者 Agent）负责协调。摄影师负责拍摄（专家 Agent）。录音师负责录音（专家 Agent）。他们不会同时做所有决定——有清晰的层级关系、清晰的交接，以及防止冲突的拍摄计划表。

---

## 第二步：识别盲区

| 盲区 | 我的说法 | 我不确定的地方 |
|------|---------|--------------|
| 1 | "明确定义的角色" | 角色分离在实践中是什么样子的？ |
| 2 | "协调机制" | Agent 之间如何交接工作而不丢失 context？ |
| 3 | "编排者 vs 专家" | 你如何决定什么被编排、什么被执行？ |
| 4 | "共享状态" | 多个 Agent 如何安全地共享和修改同一文件？ |

---

## 第三步：填补盲区

### 盲区1：实践中的角色分离

**解答**（[Anthropic — How we built our multi-agent research system](https://www.anthropic.com/engineering/multi-agent-research-system)）：

Anthropic 的多 Agent 研究系统按以下方式分离 Agent：
- **职责范围**——每个 Agent 拥有一个有界的领域（一个模块、一种文档类型、一种任务类型）
- **访问级别**——Agent 只拥有与其角色相关的工具
- **输出格式**——每个 Agent 产出一个结构化制品（artifact），供下一个 Agent 使用

角色结构示例：
```
编排者 Agent（Orchestrator Agent）
├── 拥有：整体任务计划、进度追踪
├── 工具：spawn_subagent, read_artifact, write_task_status
└── 不做：写代码、修改文件

实现 Agent（Implementation Agent）
├── 拥有：为指定模块编写代码
├── 工具：read_file, write_file, run_tests
└── 不做：生成子 Agent、修改其他模块

审查 Agent（Reviewer Agent）
├── 拥有：对实现制品进行质量检查
├── 工具：read_file, run_tests, write_review
└── 不做：修改文件，只读取和评论
```

**简单版本**：每个 Agent 都有岗位职责、一套工具和一条边界。Harness 负责强制执行这条边界。

---

### 盲区2：协调与交接（Handoffs）

**问题**：Agent 之间如何交接工作而不丢失 context？

**解答**（[Anthropic — Building effective agents](https://www.anthropic.com/engineering/building-effective-agents)）：

Anthropic 的模式：Agent 通过**结构化制品（structured artifacts）**进行通信，而不是通过共享的 context window。

每个 Agent：
1. 读取其输入制品（被交接给它的内容）
2. 完成其工作
3. 写出其输出制品（它交接给下一个 Agent 的内容）
4. 输出制品是*唯一*的通信渠道

这意味着：
- Agent 之间没有共享的可变状态
- 每个 Agent 用干净的 context 重新开始（加上输入制品）
- 故障是隔离的——一个 Agent 失败不会污染另一个 Agent 的 context
- 编排者可以检查任何制品以了解系统状态

**简单版本**：Agent 通过文档交流，而不是通过共享内存。每份文档是一个干净的、可检查的交接。

---

### 盲区3：编排者 vs 专家

**问题**：什么放在编排者里，什么放在专家 Agent 里？

**解答**（来自 Anthropic 的多 Agent 文章）：

| 决策 | 编排者 | 专家 |
|------|--------|------|
| 下一步做什么 | ✅ | ❌ |
| 如何完成具体任务 | ❌ | ✅ |
| 任务是否完成 | ✅（验证） | ✅（自我报告） |
| 执行工具 | ❌ | ✅ |
| 任务规划 | ✅ | ❌ |
| 错误恢复 | ✅（重试/重新分配） | ✅（任务内部） |

编排者是**规划者和协调者**。它不应该执行。专家是**执行者**。它们不应该规划超出分配任务之外的内容。

**简单版本**：编排者 = 项目经理。专家 = 工程师。不要让 PM 写代码，不要让工程师管理路线图。

---

### 盲区4：共享状态与冲突

**问题**：如何防止 Agent 向同一文件写入冲突的更改？

**解答**：

三种 harness 模式：

**1. 独占分配（Exclusive assignment）**——每个文件/模块在同一时间只分配给一个 Agent。编排者追踪分配情况。不能有两个 Agent 同时持有相同的分配。

**2. 制品优先工作流（Artifact-first workflow）**——Agent 写入中间制品，而不是直接写入代码库。合并步骤（由编排者或专门的合并 Agent 处理）整合更改。

**3. Git 分支隔离（Git-branch isolation）**——每个 Agent 在自己的分支上工作。编排者在每个 Agent 完成后管理合并。冲突被显式解决，而不是被静默覆盖。

**简单版本**：与人类团队避免合并冲突的方式相同——检出你的分支，在隔离中工作，谨慎合并。Agent 需要同样的纪律。

---

## 第四步：精炼解释

### 三种多 Agent 模式

**模式1：顺序管道（Sequential Pipeline）**
```
任务 → Agent A → 制品A → Agent B → 制品B → Agent C → 完成
```
- 最容易推理
- 每个 Agent 从前一个制品获得完整 context
- 瓶颈：一个失败的 Agent 会阻塞整个管道

**模式2：并行扇出（Parallel Fan-Out）**
```
任务 → 编排者 → [Agent A, Agent B, Agent C] → 合并 → 完成
```
- 对于可并行化的工作来说最快
- 需要一个能整合潜在冲突输出的合并步骤
- 最适合："每个 Agent 处理一个不同的模块"

**模式3：层级结构（Hierarchical / Tree）**
```
编排者
├── 子编排者A → [Agent A1, Agent A2]
└── 子编排者B → [Agent B1, Agent B2]
```
- 可扩展到非常大的任务
- 每个层级管理自己的范围
- 复杂性迅速增加——只有在并行性必不可少时才使用

### 多 Agent 何时有价值（何时没有）

✅ **在以下情况使用多 Agent**：
- 任务是真正可并行化的（独立模块、独立文档）
- 每个子任务复杂到需要自己的 context window
- 出于安全原因，不同角色需要不同的工具访问权限

❌ **在以下情况不使用多 Agent**：
- 任务是顺序的且相互依赖的
- 协调开销超过并行化收益
- 一个具有良好 context 管理的单一 Agent 就足够了

**经验法则**：从一个 Agent 开始。当遇到具体瓶颈时（context 溢出、需要并行性、安全边界）再添加 Agent。

### 核心要点

1. Agent 通过结构化制品通信，而不是共享 context。
2. 每个 Agent 有明确的角色、有界的范围和有限的工具访问权限。
3. 编排者规划和协调——它不执行。
4. 从单一 Agent 开始。只有在有特定原因时才添加 Agent。

---

### 30秒电梯演讲

> "多 Agent 系统放大 Agent 的能力——但前提是每个 Agent 有清晰的角色分离、有界的范围和结构化的交接。没有这些，Agent 相互争斗而不是相互帮助。"

---

## 🧪 实践 8

**设计练习**：为以下任务设计一个多 Agent harness：*"审计一个代码库的安全漏洞并产出修复方案。"*

**A部分——角色设计**：定义3-4个 Agent 角色。对于每个角色，明确：
- 它拥有什么
- 它有哪些工具
- 它**不**做什么

**B部分——制品链**：在 Agent 之间流动的结构化制品是什么？描述每个交接文档的格式/内容。

**C部分——冲突预防**：两个 Agent 可能都发现了同一个漏洞并提出了不同的修复方案。你的 harness 设计如何防止产生冲突的修复计划？

**D部分——何时并行**：这次审计的哪些部分可以并行运行？哪些部分必须是顺序的？画一个简单的依赖关系图。

---

## 延伸阅读

- [How we built our multi-agent research system](https://www.anthropic.com/engineering/multi-agent-research-system) — Anthropic 的角色分离和结构化协调架构
- [Building effective agents](https://www.anthropic.com/engineering/building-effective-agents) — Anthropic 关于工作流、Agent 和结构化系统何时胜出的指南
- [MAgIC benchmark](https://zhiyuanhubj.github.io/MAgIC/) — 衡量多 Agent 系统中的认知和协作能力
- [deepagents](https://github.com/langchain-ai/deepagents) — LangChain 用于长时间运行多 Agent 系统的工具集

---

*← [第7讲：Runtimes 与参考实现](./lecture-07-runtimes-and-implementations.md)*
*→ [第9讲：工具设计](./lecture-09-tool-design.md)*
