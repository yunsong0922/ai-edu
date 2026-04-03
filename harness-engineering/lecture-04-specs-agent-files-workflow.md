# 第4讲：Specs、Agent Files 与工作流设计

> **费曼核心概念**：如何给 Agent 提供跨会话持久存在、可在团队间规模化使用的结构化指令。

---

## 第一步：简单解释

你雇了一位承包商来装修厨房。你不会整天跟在他们后面解释你的需求。你会递给他们一份**规格说明书（spec sheet）**：尺寸、材料、风格，以及**不能动**的地方。

当 Agent 在代码库中工作时，他们需要同样的东西——一份随时可以查阅的持久文档。没有它，每次会话都要从零开始。Agent 不知道你的编码规范、哪些文件敏感、或者你偏好的测试方式。

**Agent files**（`CLAUDE.md`、`AGENTS.md`、`agent.md`）就是那份规格说明书。它们存在于代码仓库中，跨会话持续存在，告诉 Agent"我们在这里是这样做事的。"

### 类比 📋

员工手册 + 岗位职责说明：岗位职责告诉你*做什么*。员工手册告诉你*我们在这里怎么做事*——文化、规则，以及去年因为踩了某条线被解雇的教训。Agent files 兼具两者。

---

## 第二步：识别盲区

| 盲区 | 我的说法 | 我不确定的地方 |
|------|---------|--------------|
| 1 | "跨会话持久存在" | Agent files 是如何被实际加载的？谁来读取它们？ |
| 2 | "CLAUDE.md vs AGENTS.md vs agent.md" | 这几种格式实际有什么区别？ |
| 3 | "12-Factor Agents" | 这些原则是什么，为什么重要？ |
| 4 | "Spec-driven development（规格驱动开发）" | Specs 如何改变 Agent 工作流，与临时 prompts 相比有何不同？ |

---

## 第三步：填补盲区

### 盲区1：Agent Files 如何被加载

**问题**：Agent 什么时候实际读取 `CLAUDE.md`？是自动的吗？

**解答**（[HumanLayer — Writing a good CLAUDE.md](https://www.humanlayer.dev/blog/writing-a-good-claude-md)）：

Claude Code 会在每次会话开始时自动读取 `CLAUDE.md`（如果它存在于工作目录或任何父目录中）。Harness 也可以指示 Agent 在任务开始时读取特定领域的文件——例如，"在触碰任何模块之前，先读取 ARCHITECTURE.md"。

该文件必须满足：
1. **存在于代码仓库中**——不是外部的，不是临时的
2. **使用纯 Markdown 格式**——Agent 像读文档一样读取它
3. **被 harness 信任**——不是随便一个文件，而是 harness 明确加载的文件

**简单版本**：它就像 Agent 的 `.bashrc`——在每次会话启动时自动加载。

---

### 盲区2：CLAUDE.md vs AGENTS.md vs agent.md

**问题**：这三种格式都存在，它们实际上有什么区别？

**解答**：

| 格式 | 来源 | 适用范围 | 关键区别 |
|------|------|---------|---------|
| `CLAUDE.md` | Anthropic | Claude Code 专用 | 工具特定，在 CC 中有丰富的支持 |
| `AGENTS.md` | [agentsmd/agents.md](https://github.com/agentsmd/agents.md) | 开放标准，工具无关 | 兼容 Claude、Codex、Cursor 等 |
| `agent.md` | [agentmd/agent.md](https://github.com/agentmd/agent.md) | 相关开放标准 | 略有不同的 schema，目标相同 |

实践中：选一个，放在代码仓库根目录，保持一致。内容比文件名更重要。

**简单版本**：同一种语言的不同方言——"这是在这个代码库里工作的方式。"

---

### 盲区3：12-Factor Agents（十二要素 Agent）

**问题**：十二要素 Agent 原则是什么？

**解答**（[HumanLayer — 12-Factor Agents](https://www.humanlayer.dev/blog/12-factor-agents)）：

模仿 Web 服务的十二要素应用宣言。Agent harness 设计的关键原则：

| 要素 | 原则 | 含义 |
|------|------|------|
| 显式 prompts（Explicit prompts） | 无隐藏上下文 | Agent 收到的每条指令都是可见且可审计的 |
| 无状态 Agent（Stateless agents） | Harness 拥有状态 | Agent 不在多次运行间积累状态——harness 负责此事 |
| 声明式工具（Declarative tools） | 将工具视为函数 | 工具尽量是无状态、无副作用的 |
| 暂停-恢复（Pause-resume） | 干净的中断 | Agent 可以被停止并从已知检查点（checkpoint）恢复 |
| 关注点分离（Separation of concerns） | 逻辑 vs 管道 | Agent 逻辑与运行时基础设施（重试、日志）分离 |
| 可观测（Observable） | 追踪一切 | 每个 Agent 动作都被记录并可检查 |

**简单版本**：就像好的微服务——无状态、显式、可恢复、可观测。

---

### 盲区4：Spec-Driven Development（规格驱动开发）

**问题**：在 Agent 开始工作之前写一份 spec，会如何改变结果？

**解答**（[Thoughtworks — Understanding Spec-Driven-Development](https://martinfowler.com/articles/exploring-gen-ai/sdd-3-tools.html) + [GitHub Spec Kit](https://github.com/github/spec-kit)）：

没有 spec，Agent 在每个决策点都会模糊地解读目标。细微的误解不断积累。最终你需要 review 几百行"差不多正确"的代码。

有了 spec，Agent 有了一个明确的参照来验证自己的工作。每个决策点变成："这是否满足 spec？"而不是"这感觉对吗？"

[12-Factor AgentOps](https://www.12factoragentops.com/) 增加了一个运营视角：spec 还让 Agent 工作流**可重现**——如果你能用同一份 spec 运行两次并获得相同结果，就能调试、改进和 A/B 测试你的 harness。

**简单版本**：Specs 就像 Agent 理解力的单元测试——它们让"正确"变得具体可衡量。

---

## 第四步：精炼解释

### 什么是好的 Agent File

```markdown
# CLAUDE.md — 示例结构

## 这个代码库是做什么的
一个面向电商订单管理的 REST API。
使用 Node.js + Express + PostgreSQL。

## 架构
- /src/routes/        — HTTP 路由处理器（薄层，委托给 service 层）
- /src/services/      — 业务逻辑（此处不直接访问 DB）
- /src/repositories/  — 数据库层（通过 pg 库访问 Postgres）
- /src/models/        — 类型定义（TypeScript interfaces）

## 规范约定
- 数据库访问始终使用 Repository 模式（路由层中不能直接查询）
- 所有异步函数必须有显式的错误处理（不允许静默失败）
- 新 endpoint 需要在 /tests/integration/ 中有集成测试

## 禁止触碰
- /src/legacy/        — 已废弃，迁移进行中（任何改动前先咨询）
- .env 文件           — 绝不读取、修改或记录这些文件

## 测试
标记任务完成前必须运行：
  npm test              （单元测试——必须全部通过）
  npm run test:int      （集成测试——必须全部通过）

## Handoff（交接）
完成任务时，创建 TASK_STATUS.md，包含：
- 完成了什么
- 运行了哪些测试
- 什么尚未完成
- 做了哪些决策及其原因
```

### Agent File 成熟度阶梯

| 级别 | 现有内容 | Agent 行为 |
|------|---------|-----------|
| 0 — 无 | 没有 agent file | Agent 猜测规范，做出不一致的选择 |
| 1 — 基础 | 列出代码库的功能 | 减少对代码库的错误假设 |
| 2 — 规范约定 | 添加 do/don't 规则 | 风格和模式保持一致 |
| 3 — 边界 | 添加"禁止触碰"列表 | 避开敏感区域 |
| 4 — 验证 | 添加测试命令 | 在宣告完成前自我验证 |
| 5 — 交接 | 添加制品（artifact）规范 | 会话可以干净地恢复 |

目标：达到第4级及以上。

### 核心要点

1. Agent files 是 harness 的持久记忆——它们在每次会话中都能幸存。
2. 好的 agent files 明确指出**不**该做什么，而不仅仅是该做什么。
3. Specs 让 Agent 工作流确定且可重现——无歧义，无累积的误解。
4. 将 agent files 视为活文档——当 Agent 犯了出人意料的错误时，更新它。

---

### 30秒电梯演讲

> "Agent files 是代码仓库中的持久指令。没有它们，每次会话从零开始，规范不断漂移。有了它们，Agent 从第一行就知道你的标准。"

---

## 🧪 实践 4

**写作练习**：为一个假想项目编写一份 `CLAUDE.md`——一个小型电商店铺的 REST API。

包含以下**所有**章节：

1. **代码库是做什么的** — 最多2句话
2. **架构** — 列出关键文件夹/模块及各自的职责
3. **规范约定** — Agent 必须遵守的2条规则（例如：测试模式、错误处理）
4. **禁止触碰** — 1个未经人工批准不可修改的区域
5. **测试** — 标记任务完成前需要运行的精确命令
6. **交接（Handoff）** — Agent 完成后必须留下什么内容

**加分项**：找出一件你的代码库中一个"天真"的 Agent 可能犯错、但你的 `CLAUDE.md` 能防止的事情。

---

## 延伸阅读

- [AGENTS.md 开放格式](https://github.com/agentsmd/agents.md) — 跨工具的机器可读 Agent 指令
- [Writing a good CLAUDE.md](https://www.humanlayer.dev/blog/writing-a-good-claude-md) — 持久化代码仓库本地指令的实践指南
- [12 Factor Agents](https://www.humanlayer.dev/blog/12-factor-agents) — 生产级 Agent 的操作原则
- [12-Factor AgentOps](https://www.12factoragentops.com/) — 上下文纪律、验证、可重现工作流
- [GitHub Spec Kit](https://github.com/github/spec-kit) — Spec-driven development 工具集
- [Understanding Spec-Driven-Development](https://martinfowler.com/articles/exploring-gen-ai/sdd-3-tools.html) — Thoughtworks 谈强力 specs 如何让 AI 交付更可靠
- [Claude Code: Best practices for agentic coding](https://code.claude.com/docs) — Anthropic 的实践建议

---

*← [第3讲：约束与安全自主性](./lecture-03-constraints-and-safe-autonomy.md)*
*→ [第5讲：Evals 与可观测性](./lecture-05-evals-and-observability.md)*
