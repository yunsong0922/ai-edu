# 第1讲：什么是 Harness（驾具）？

> **Feynman 概念**：Harness 的基础理念，以及为什么它比模型本身更重要。

---

## 第一步：简单解释

想象你雇了一个天才但极度字面理解的实习生。他能写代码、做调研、回答问题——但他不了解*你的*公司、*你的*标准，也不了解*你的*代码库。如果你只说"修复登录 bug"，他可能会把整个认证系统从头重写一遍。

**Harness（驾具）** 是你围绕这个实习生所做的一切，让他能真正发挥作用：
- 你交给他的入职文档
- 能捕获他错误的 linter（代码检查工具）
- "不经审查不能动数据库"这条规则
- 提交前必须通过的测试套件
- 他每天下班前留下的结构化交接记录，让明天的班次可以继续

**Harness 不是实习生（模型）本身。它是让实习生可靠的脚手架（scaffolding）。**

### 类比 🐕

训练一只狗：狗（模型）可以非常聪明。但没有牵引绳（guardrails 护栏）、训练指令（prompts 提示词）、好行为奖励（evals 评估），以及每天回到同一个公园（context 一致性）——即使最聪明的狗也会乱跑。

---

## 第二步：识别盲区

| 盲区 | 我说的 | 我不确定的 |
|------|--------|-----------|
| 1 | "模型周围的脚手架" | 到底什么算 harness，什么算 framework，什么算 runtime？ |
| 2 | "让 Agent 可靠" | AI Agent 的"可靠"到底是什么意思？ |
| 3 | "Harness 不是模型" | 但更好的模型难道不能减少对 harness 的需求吗？ |

### 术语自查

| 术语 | 能简单解释吗？ |
|------|--------------|
| Harness（驾具） | 部分——"模型周围的环境" |
| Scaffolding（脚手架） | 能——临时支撑结构 |
| Context window（上下文窗口） | 不能 → 见第2讲 |

---

## 第三步：填补盲区

### 盲区 1：Harness vs Framework vs Runtime

**问题**：LangChain 被称为"framework（框架）"，但也做 harness 的事情。区别在哪？

**回答**（[LangChain anatomy 文章](https://blog.langchain.com/the-anatomy-of-an-agent-harness/)）：
- **Framework（框架）** = 代码库（LangChain、LlamaIndex）
- **Runtime（运行时）** = 执行环境——状态、重试、并发、持久化
- **Harness（驾具）** = 围绕 Agent 的*设计*——提示词、工具、约束、评估逻辑

你用 framework 来*构建* harness，两者不是同一回事。

**简单版**：Framework = 锤子。Runtime = 工作间。Harness = 建造蓝图。

---

### 盲区 2："可靠"是什么意思？

**问题**：可靠地做什么？

**回答**（[Anthropic — Effective Harnesses](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents)）：

可靠意味着：
- 完成长任务时不偏离原始目标
- 跨 context window 边界时正确交接
- 在宣布完成之前自我验证输出
- 留下制品（artifacts），让人类或其他 Agent 能从中断处继续

**简单版**：可靠 = "即使事情变复杂，它也能做你想让它做的事。"

---

### 盲区 3：更好的模型能替代 Harness 吗？

**回答**（[HumanLayer — Skill Issue](https://www.humanlayer.dev/blog/skill-issue-harness-engineering-for-coding-agents)）：

AI 编程 Agent 的弱表现通常是 **harness 问题**，而不是模型问题。更好的模型配上烂 harness 照样失败。Harness 质量的提升可以超过整个模型档次的差距。

证据：LangChain 证明了[仅靠 harness 改动就能显著提升 benchmark 分数](https://blog.langchain.com/improving-deep-agents-with-harness-engineering/)，与所用模型无关。

**简单版**：就算是世界上最好的司机，在没有护栏的山路上也会翻车。

---

## 第四步：精炼解释

### 最终简单解释

一个 AI Agent 需要的不只是聪明的大脑，还需要：

1. **清晰的指令**——在*这个特定*代码库中该做什么、如何行为
2. **记忆管理**——在漫长任务中记住什么
3. **护栏（Guardrails）**——永远不能做什么
4. **接口清晰的工具**——让它能正确使用
5. **验证（Verification）**——我们如何知道它做好了

这些合在一起，就是 **harness**。Harness 是环境，不是模型。

> Harness 质量与模型质量*独立*。优秀的 harness 能让普通模型变好；糟糕的 harness 会浪费优秀模型。

### 三层架构图

```
┌──────────────────────────────────────────┐
│              HARNESS（驾具）              │
│  ┌──────────────────────────────────┐    │
│  │  指令 / Agent Files              │    │
│  │  记忆 / Context Management       │    │
│  │  工具 / Interfaces               │    │
│  │  约束 / Guardrails               │    │
│  │  评估 / Verification             │    │
│  └──────────────────────────────────┘    │
│                                          │
│           ┌─────────┐                   │
│           │  MODEL  │  ← 只是其中一部分  │
│           └─────────┘                   │
└──────────────────────────────────────────┘
```

### 核心要点

1. 模型是发动机。Harness 是汽车、道路和交通规则的总和。
2. 大多数 Agent 失败是 harness 失败，而不是模型失败。
3. Harness 工作 = context engineering（上下文工程）+ 架构约束 + 评估。

---

### 30 秒电梯演讲

> "Harness 是 AI 周围的一切——指令、规则、记忆、工具和检查——让它的行为可预测。更好的 harness 往往胜过更好的模型。"

---

## 🧪 实践练习 1

**思想实验**：你想用一个 AI Agent 自动审查并合并 GitHub 仓库中的 Pull Request。

写下这四个问题的答案：

1. **指令**——你会在 `CLAUDE.md` 或 `AGENTS.md` 里写什么？Agent 需要提前知道什么？

2. **护栏（Guardrails）**——它绝对不允许做什么？（例如：直接合并到 main？关闭 issue？强制 push？）

3. **评估（Evaluation）**——你如何判断它做得*好*还是*差*？你会衡量什么？

4. **交接制品（Handoff artifact）**——它完成后应该留下什么，让人类能够验证工作？

> 💡 提示：如果你无法回答全部四个问题，这正是重点所在。那些空白 = 你的 harness 设计问题。

---

## 延伸阅读

- [Harness engineering: leveraging Codex in an agent-first world](https://openai.com/index/harness-engineering/) — OpenAI 旗舰实战报告
- [Effective harnesses for long-running agents](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents) — Anthropic 核心文章
- [The Anatomy of an Agent Harness](https://blog.langchain.com/the-anatomy-of-an-agent-harness/) — LangChain 简洁框架
- [Skill Issue: Harness Engineering for Coding Agents](https://www.humanlayer.dev/blog/skill-issue-harness-engineering-for-coding-agents) — 弱结果通常是 harness 问题
- [Your Agent Needs a Harness, Not a Framework](https://www.inngest.com/blog/your-agent-needs-a-harness-not-a-framework) — 状态、重试、追踪作为一等基础设施

---

*下一讲：[第2讲 → Context 与记忆](./lecture-02-context-and-memory.md)*
