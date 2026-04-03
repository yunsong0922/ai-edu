# 第10讲：Context Engineering 进阶模式

> **费曼核心概念**：超越"管理你的 context window"——在生产规模下处理 context 的工程学科和具体技术。

---

## 第一步：简单解释

第2讲介绍了基础知识：context window 就像白板，保留重要的便签，把其余的摘要化。

但当你为真实的生产工作负载构建真实的 harness 时，"摘要化其余的"不是工程规范，它只是一个愿望。

进阶 context engineering 提出了更难的问题：
- *什么时候*触发压缩？
- *如何*在不丢失关键信息的情况下压缩？
- 当 Agent 产生的噪声输出比有用工作更快地填满 context window 时，*怎么办*？
- 对于跨越数天的任务，*如何*保持 context 的"新鲜度"？

### 类比 💻

计算机的虚拟内存系统：当 RAM 满了，操作系统不会随机删除程序。它有精确的策略——它识别出最近没有被访问过的内存页（LRU，最近最少使用），将它们换页到磁盘，并将常用页保留在 RAM 中。需要时再换页回来。进阶 context engineering 也是如此：深思熟虑的、基于策略的、可逆的。

---

## 第二步：识别盲区

| 盲区 | 我的说法 | 我不确定的地方 |
|------|---------|--------------|
| 1 | "在80%时触发压缩" | 正确的阈值是什么？什么触发它？ |
| 2 | "安全地压缩而不丢失关键信息" | 你实际上如何安全地压缩？ |
| 3 | "背压（Backpressure）" | 这个术语在 context engineering 中意味着什么？ |
| 4 | "工具屏蔽（Tool masking）" | 什么是工具屏蔽，何时使用？ |

---

## 第三步：填补盲区

### 盲区1：压缩触发条件和阈值

**解答**（[OpenHands — Context Condensation](https://openhands.dev/blog/openhands-context-condensensation-for-more-efficient-ai-agents)）：

OpenHands 的生产系统使用**有界对话记忆（bounded conversation memory）**设计：

在以下情况触发压缩：
- Context 达到阈值（例如，最大 window 的75-80%）
- 或达到自然断点（任务阶段边界、工具调用批次完成）

阈值不是关于单一的魔法数字。核心洞见：在*需要*压缩之前触发压缩，而不是在到达极限时。在极限时，你已经没有空间来完成压缩工作本身了。

**压缩策略**：
```
始终保留（never compress）：
  - 当前任务目标
  - 当前任务约束（来自 agent file）
  - 正在修改的文件列表
  - 失败的测试名称 + 错误信息
  - 最近2-3轮（近期上下文）

摘要化（compress to ~10% of original）：
  - 已完成的工具调用序列
  - 超出近期窗口的前几轮
  - 对稳定文件的读取（缓存摘要）

完全丢弃（Drop entirely）：
  - 没有持久相关性的成功工具调用
  - 已处理过的诊断/调试输出
  - 重复内容（同一文件读取多次）
```

**简单版本**：主动压缩，而非被动响应。你最不想要的，就是在关键推理步骤的中途压缩。

---

### 盲区2：安全压缩技术

**解答**（[Manus — Context Engineering for AI Agents](https://manus.im/blog/Context-Engineering-for-AI-Agents-Lessons-from-Building-Manus) + [HumanLayer — Advanced Context Engineering](https://www.humanlayer.dev/blog/advanced-context-engineering)）：

**技术1：渐进式摘要（Progressive summarization）**
不要一次压缩所有内容。分层摘要：
- 近期轮次：逐字保留
- 中期轮次：在句子层面摘要
- 旧轮次：在段落层面摘要
- 非常老的轮次：在"已完成工作"日志中只占一行

**技术2：持久化文件系统记忆（Persistent filesystem memory）**
将信息从 context 移到文件中：
```
不是：在 context 中保留500行文件分析
而是：将分析写入 /task/notes/auth-module-analysis.md
      在 context 中保留："分析已写入 /task/notes/auth-module-analysis.md — 关键发现：第247行的废弃 API"
```

如果需要，Agent 可以重新读取该文件，但它不会持续占用 context。

**技术3：KV 缓存局部性保持（KV-cache locality preservation）**
压缩时，保持结构稳定：
- 系统 prompt：始终在位置0，永不修改
- Agent file 内容：始终在位置1，永不修改
- 压缩后的历史：始终在相同的结构槽位
- 活跃的工作 context：始终在末尾

在位置之间移动内容会破坏 KV 缓存并增加成本。

**简单版本**：增量压缩，尽可能外化到文件，保持 context 结构稳定以使缓存正常工作。

---

### 盲区3：背压（Backpressure）

**问题**：在 Agent context 中，背压是什么？

**解答**（[HumanLayer — Context-Efficient Backpressure](https://www.humanlayer.dev/blog/context-efficient-backpressure)）：

在分布式系统中，背压是当消费者跟不上时减慢生产者速度的机制。在 context engineering 中，背压是防止 Agent 产生比它能有效使用的更多 context 的 harness 机制。

需要背压的症状：
- Agent 在进行探索性搜索，填满了 context 但没有对任务做出贡献
- 工具调用返回大量输出，Agent 读取了但没有使用
- Agent 生成了冗长的推理但没有导向行动

背压技术：
1. **工具输出上限（Tool output caps）**——工具最多返回N行；如果需要更多，Agent 必须明确请求
2. **搜索结果过滤（Search result filtering）**——不要返回100条搜索结果；返回10条带摘要的结果
3. **强制检查点（Forced checkpoints）**——在N次工具调用后，Agent 必须在继续之前摘要进度
4. **Token 预算信号（Token budget signals）**——明确告诉 Agent 剩余多少 context 预算

**简单版本**：如果你的 Agent 在噪声上消耗 context，harness 需要减慢噪声产生速度，而不仅仅是事后清理。

---

### 盲区4：工具屏蔽（Tool Masking）

**问题**：什么是工具屏蔽？

**解答**（[Manus — Context Engineering](https://manus.im/blog/Context-Engineering-for-AI-Agents-Lessons-from-Building-Manus)）：

工具屏蔽是在当前任务阶段与工具不相关时，将工具从 Agent 的 context 中隐藏的实践。

示例：
```
阶段：规划（Planning）
  可用工具：read_file, list_directory, search_code
  屏蔽工具：write_file, run_tests, deploy（从 context 中隐藏）

阶段：实现（Implementation）
  可用工具：read_file, write_file, run_tests
  屏蔽工具：deploy, send_email（从 context 中隐藏）

阶段：验证（Verification）
  可用工具：run_tests, read_file
  屏蔽工具：write_file, deploy（从 context 中隐藏）
```

好处：
- 减少 Agent 推理的 context 空间
- 防止 Agent 采取过早的行动（无法调用不可见的工具）
- 缩小决策空间 → 在可用空间中做出更好的决策

**简单版本**：当 Agent 还在写代码时，不要把"部署"按钮放在屏幕上。

---

## 第四步：精炼解释

### 进阶 Context Engineering 技术栈

```
第1级 — 基础（第2讲）：
  保留重要内容，摘要其余的，交接制品

第2级 — 中级：
  主动压缩，文件系统记忆，结构化槽位

第3级 — 进阶（本讲）：
  KV 缓存局部性，工具屏蔽，背压，阶段感知 context

第4级 — 生产级：
  自动化压缩管道，context 质量指标，
  A/B 测试 context 策略，压缩 eval 套件
```

### Context 作为一等资源

生产级 harness 将 context 视为金钱——每个 token 都有成本，你要刻意制定预算：

```
Context 预算：200k tokens

固定成本（不可协商）：
  系统 prompt：           3k tokens
  Agent file（CLAUDE.md）：2k tokens
  当前任务目标：          0.5k tokens
  固定成本合计：          5.5k tokens

可变预算：194.5k tokens 剩余

分配：
  活跃工作 context：      40k  (20%)
  工具结果（过滤后）：    60k  (30%)
  压缩后的历史：          94.5k (50%)
```

当你接近预算限制时，先压缩最容易压缩的东西（旧历史），而不是最重要的东西。

### 核心要点

1. 在70-80%时主动压缩，而不是在100%时被动响应。
2. 将稳定信息外化到文件——context 用于活跃推理，不用于存储。
3. 工具屏蔽限制了决策空间——Agent 在选项更少时做出更好的决策。
4. 背压防止 context 泛滥——在源头减慢噪声产生。

---

### 30秒电梯演讲

> "进阶 context engineering 是 AI 的缓存管理。刻意制定预算，主动压缩，外化到文件，屏蔽无关工具，在 window 泛滥前施加背压。"

---

## 🧪 实践 10

**工程练习**：

**A部分——压缩策略**：为正在重构大型代码库的 Agent 编写具体的压缩策略。要具体说明：
- 什么触发压缩（阈值 + 条件）？
- 什么**始终**逐字保留？
- 什么被摘要化（如何——每X行一句话）？
- 什么被完全丢弃？

**B部分——背压场景**：你的 Agent 正在搜索代码库中某个已废弃函数的用法。它运行了15次搜索，每次返回200行结果。搜索完成时，它已经用了40%的 context window 来存放可能不会再重读的搜索结果。

设计一个背压机制：你将如何改变工具或 harness 来防止这种情况？

**C部分——工具屏蔽设计**：为"编写新功能"任务设计一个3阶段的工具屏蔽策略：
- 阶段1：理解（读取和规划）
- 阶段2：实现（编写代码）
- 阶段3：验证（测试和审查）

对于每个阶段，列出哪些工具可见，哪些被屏蔽。

---

## 延伸阅读

- [Effective context engineering for AI agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents) — Anthropic 将 context window 视为工作记忆预算
- [Context Engineering for AI Agents: Lessons from Building Manus](https://manus.im/blog/Context-Engineering-for-AI-Agents-Lessons-from-Building-Manus) — KV 缓存局部性、工具屏蔽、文件系统记忆
- [OpenHands Context Condensation](https://openhands.dev/blog/openhands-context-condensensation-for-more-efficient-ai-agents) — 有界对话记忆设计
- [Advanced Context Engineering for Coding Agents](https://www.humanlayer.dev/blog/advanced-context-engineering) — 减少 context 漂移
- [Context-Efficient Backpressure for Coding Agents](https://www.humanlayer.dev/blog/context-efficient-backpressure) — 防止低价值 context 消耗
- [Context Engineering for Coding Agents (Thoughtworks)](https://martinfowler.com/articles/exploring-gen-ai/context-engineering-coding-agents.html) — 塑造任务环境

---

*← [第9讲：工具设计](./lecture-09-tool-design.md)*
*→ [第11讲：长任务应用](./lecture-11-long-running-apps.md)*
