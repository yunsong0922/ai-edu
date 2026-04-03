# 第2讲：Context（上下文）与记忆——Agent 的工作记忆

> **Feynman 概念**：AI Agent 如何在漫长任务中管理信息，以及为什么这是一个难啃的工程问题。

---

## 第一步：简单解释

想象你正在做一项复杂任务——重写一个大型代码库中的某个模块。你开始读文件，又读更多文件，写了一些代码，跑了测试。但你脑子里能装的东西是有限的。

AI Agent 面临同样的问题。它们有一个 **context window（上下文窗口）**——固定大小的"工作记忆"。想象一块只能贴 100 张便签的白板。你贴新便签时，旧的就会从边缘掉下去。

如果掉下去的是错误的便签——比如"不要用废弃的 API X"或"用户真正的目标是 Y"——Agent 就会搞坏不该搞的东西。

**Context engineering（上下文工程）** 就是决策的艺术：*哪些便签贴上去，什么时候贴？*

### 类比 🧠

12 小时手术中的外科医生：他们无法同时记住所有事情。他们依赖器械护士（context 管理）在正确时机递上正确工具，追踪纱布（工作状态），记录一切（制品）。没有那个护士，即使最好的外科医生也会出错。

---

## 第二步：识别盲区

| 盲区 | 我说的 | 我不确定的 |
|------|--------|-----------|
| 1 | "固定的工作记忆" | KV-cache 局部性（KV-cache locality）在实践中有什么影响？ |
| 2 | "错误的便签掉下去" | 如何决定*保留*什么在 context 中？ |
| 3 | "Context 漂移（Context drift）" | Context 漂移到底是什么，如何累积？ |
| 4 | "恢复长会话" | Harness 如何帮助 Agent 从中断处继续？ |

---

## 第三步：填补盲区

### 盲区 1：KV-Cache 局部性（KV-Cache Locality）

**问题**：把东西放在 context 的*哪里*，实际影响有多大？

**回答**（[Manus — Context Engineering for AI Agents](https://manus.im/blog/Context-Engineering-for-AI-Agents-Lessons-from-Building-Manus)）：

推理引擎会缓存它之前见过的 token 的"键值"（key-value）计算。如果你的 system prompt 始终不变且每次都放在开头，引擎就会缓存并低成本复用。如果你随意挪动内容，就会破坏缓存，每次都要付全额计算成本。

**简单版**：把最常引用的内容（system prompt、核心指令）放在**最顶部**，永远不移动它。就像 CPU 缓存——稳定的数据 = 快速。

---

### 盲区 2：保留什么在 Context 中

**问题**：窗口 80% 满了，该丢掉什么？

**回答**（[OpenHands — Context Condensation](https://openhands.dev/blog/openhands-context-condensensation-for-more-efficient-ai-agents)）：

OpenHands 的设计在压缩时精确保留四件事：
1. **目标（Goals）**——Agent 要完成什么
2. **进展（Progress）**——已经做了什么（摘要形式）
3. **关键文件路径（Critical file paths）**——正在修改的核心文件
4. **失败的测试（Failing tests）**——还没通过的测试

其他一切——冗长的工具输出、中间步骤、重复的文件读取——全部压缩或丢弃。

**简单版**：保留地图、当前位置，以及那个你绝不能越过的警示标志。丢掉你已经走过的路的风景描述。

---

### 盲区 3：Context 漂移（Context Drift）

**问题**：Context 漂移是什么，为什么重要？

**回答**（[HumanLayer — Advanced Context Engineering](https://www.humanlayer.dev/blog/advanced-context-engineering)）：

Context 漂移是 Agent 的"心理模型"与现实之间逐渐积累的错位。原因包括：
- 噪声工具输出填满窗口，夹杂无关细节
- 相互矛盾的指令在多轮对话中累积
- Agent "忘记"早期约束，因为它们滚出了视野

它会复利累积：每次漂移让下一个响应稍差，产生更多噪声，进而更多漂移。

**简单版**：就像传话游戏——每次传递都略微扭曲消息。长 Agent 会话会累积漂移，除非 harness 主动重置或重新注入基础事实。

---

### 盲区 4：恢复长会话

**问题**：Agent 如何在跨越 context window 边界后继续多小时的任务？

**回答**（[Anthropic — Effective Harnesses](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents)）：

Anthropic 的模式：在触碰 context 上限之前，Agent 创建一份**交接制品（handoff artifact）**——一份结构化文档，摘要：
- 完成了什么
- 剩余什么
- 文件和测试的当前状态
- 任何待解决的问题或阻碍

下一个 context 窗口（或下一个 Agent）读取这个制品来继续，无需重新阅读全部历史。

**简单版**：就像医生写交班记录，让夜班不用从头再听一遍汇报。

---

## 第四步：精炼解释

### 三大记忆问题

| 问题 | 发生什么 | Harness 解决方案 |
|------|---------|----------------|
| **溢出（Overflow）** | Context 窗口满了，早期信息丢失 | 摘要并压缩旧 context |
| **漂移（Drift）** | Agent "忘记"早期约束 | 定期重新注入关键不变量 |
| **噪声（Noise）** | 无用工具输出挤占有用信息 | 过滤并摘要工具结果后再加入 context |

### Context 预算思维

把 context 窗口想成 **RAM 预算**，而不是日志文件：

```
Context Window（例如 200k tokens）
├── System prompt / agent file     [~5%]  ← 永不移动（缓存这个）
├── 任务目标 + 约束                 [~5%]  ← 接近上限时重新注入
├── 工作状态（当前任务）            [~20%] ← 活跃，频繁更新
├── 工具结果（过滤后）              [~30%] ← 积极摘要
└── 对话历史                       [~40%] ← 压缩旧轮次
```

### 最终简单解释

设计良好的 harness 把 context 窗口当作珍贵的 RAM：
1. 稳定的指令放在最顶部（推理引擎低成本缓存）
2. 主动摘要旧轮次，而不是让它们堆积
3. 开始新任务段时重新注入关键约束
4. 在 context 边界处创建交接制品，让会话能干净地恢复
5. 过滤嘈杂的工具输出后再让其进入 context

### 核心要点

1. Context 窗口 = 空间有限的白板。要刻意管理它。
2. 是 harness，不是模型，决定什么留下，什么离开。
3. 长任务需要主动的记忆管理：摘要、压缩、重新注入、交接制品。

---

### 30 秒电梯演讲

> "Context 是稀缺的工作记忆。好的 harness 决定保留什么、摘要什么、丢弃什么——让 Agent 在漫长任务中保持专注，不漂移也不忘记原始目标。"

---

## 🧪 实践练习 2

**设计练习**：你正在为一个要在几小时内重构 10,000 行代码库的 Agent 构建 harness。

回答这四个问题：

1. **System prompt 内容**——什么放在永远不变的 system prompt 里（并受益于 KV 缓存）？

2. **压缩策略**——每处理完一个文件后，什么被摘要，什么从 context 中丢弃？

3. **必须始终保留的内容**——说出 3 件无论窗口多满都必须留在 context 里的事情。

4. **交接制品**——当 context 窗口达到 80% 时，harness 需要创建一个检查点。那个检查点文档包含什么？

> 💡 加分题：在这个重构任务中，context 漂移会是什么样子？Agent 漂移的第一个迹象是什么？

---

## 延伸阅读

- [Effective context engineering for AI agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents) — Anthropic 将 context 窗口作为工作记忆预算管理
- [Context Engineering for AI Agents: Lessons from Building Manus](https://manus.im/blog/Context-Engineering-for-AI-Agents-Lessons-from-Building-Manus) — KV-cache 局部性、工具屏蔽、文件系统记忆
- [OpenHands Context Condensation](https://openhands.dev/blog/openhands-context-condensensation-for-more-efficient-ai-agents) — 有界对话记忆设计
- [Advanced Context Engineering for Coding Agents](https://www.humanlayer.dev/blog/advanced-context-engineering) — 减少 context 漂移的模式
- [Context-Efficient Backpressure for Coding Agents](https://www.humanlayer.dev/blog/context-efficient-backpressure) — 防止 Agent 在低价值工作上消耗 context
- [Writing a good CLAUDE.md](https://www.humanlayer.dev/blog/writing-a-good-claude-md) — 持久化 repo 本地指令实用指南

---

*← [第1讲：什么是 Harness？](./lecture-01-what-is-a-harness.md)*
*→ [第3讲：约束与安全自主性](./lecture-03-constraints-and-safe-autonomy.md)*
