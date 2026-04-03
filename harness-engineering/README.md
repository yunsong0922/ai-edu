# 🎓 Harness Engineering（驾驭工程）：Feynman 系列讲义

> 基于：[walkinglabs/awesome-harness-engineering](https://github.com/walkinglabs/awesome-harness-engineering)

这是一套关于 **Harness Engineering（驾驭工程）** 的自学课程——即通过塑造 AI Agent 周围的环境，使其可靠地完成工作的实践。

学习方法采用 **Feynman 技巧（费曼学习法）**：简单解释 → 识别盲区 → 填补盲区 → 精炼解释。

---

## 什么是 Harness Engineering？

当你让一个 AI Agent 去"帮我做这个 App"时，模型（Model）只是等式的一半。另一半是模型*周围的一切*：指令、记忆、工具、护栏（guardrails）、检查点（checkpoints）。

那个"周围的一切"，就是 **harness（驾具）**。

> Harness engineering 处于 context engineering（上下文工程）、evaluation（评估）、observability（可观测性）、orchestration（编排）、safe autonomy（安全自主性）和软件架构的交叉点。

**核心洞见**：更好的 harness 胜过更好的模型——持续地、可测量地、可重现地。

---

## 课程结构（12 讲）

| # | 文件 | 主题 | 类比 | 核心洞见 |
|---|------|------|------|---------|
| 1 | [第1讲](./lecture-01-what-is-a-harness.md) | 什么是 Harness？ | 🐕 训练一只狗 | 模型只是一个组件；harness 是其他一切 |
| 2 | [第2讲](./lecture-02-context-and-memory.md) | Context 与记忆 | 🧠 贴满便签的白板 | Context window = RAM；要刻意管理 |
| 3 | [第3讲](./lecture-03-constraints-and-safe-autonomy.md) | 约束与安全自主性 | 🚗 山路护栏 | 护栏让速度更快；约束不是限制 |
| 4 | [第4讲](./lecture-04-specs-agent-files-workflow.md) | Specs、Agent Files 与工作流设计 | 📋 员工手册 | 持久化指令；CLAUDE.md 是 onboarding 文档 |
| 5 | [第5讲](./lecture-05-evals-and-observability.md) | Evals 与可观测性 | 🏥 体检 | 衡量 harness 质量，而不仅仅是模型质量 |
| 6 | [第6讲](./lecture-06-benchmarks.md) | Benchmarks（基准测试） | 🏁 赛车设计 vs. 发动机 | 同一模型，更好的 harness = 更好的成绩 |
| 7 | [第7讲](./lecture-07-runtimes-and-implementations.md) | Runtimes 与参考实现 | 🏗️ 厨房 vs. 食谱 | 持久化 runtime 是 harness 达到生产级的关键 |
| 8 | [第8讲](./lecture-08-multi-agent-systems.md) | 多 Agent 系统 | 🎭 电影剧组 | 结构化制品（artifacts）+ 角色分离 = 协调 |
| 9 | [第9讲](./lecture-09-tool-design.md) | 工具设计（Tool Design） | 🔧 良好设计的 API | 一个工具，一个动作；糟糕的接口导致系统性错误 |
| 10 | [第10讲](./lecture-10-context-engineering-advanced.md) | Context Engineering 进阶 | 💻 虚拟内存管理 | 主动压缩，屏蔽工具，施加背压（backpressure） |
| 11 | [第11讲](./lecture-11-long-running-apps.md) | 长任务应用 | 🏗️ 摩天楼建设班次 | init.sh + 功能列表 + 交接制品 = 跨会话连贯性 |
| 12 | [第12讲](./lecture-12-production-harness-capstone.md) | 生产级 Harness 综合设计 | 🏭 工厂流水线 | 所有支柱整合；成熟度模型帮你定位现状 |

---

## Harness 成熟度模型

| 级别 | 现状 | 结果 |
|------|------|------|
| 0 — 无 | 直接调用模型 API | 不一致，不可靠 |
| 1 — 基础指令 | CLAUDE.md + 基础工具 | 更一致 |
| 2 — Context 管理 | 压缩 + 交接制品 | 长任务可行 |
| 3 — 已评估 | Evals + trace 日志 + 自我验证 | 质量可衡量 |
| 4 — 受控自主 | 权限矩阵 + 沙盒 + 人工检查点 | 可用于有真实副作用的任务 |
| 5 — 生产级 | 以上全部 + 持久化执行 + 多 Agent + 基准测试 | 规模化可靠 |

---

## 如何使用本课程

每讲遵循 **Feynman 格式**：

1. **第一步 — 简单解释**：用大白话，不用术语。让一个聪明的 12 岁小孩也能听懂。
2. **第二步 — 识别盲区**：诚实地列出哪些地方还模糊、不确定。
3. **第三步 — 填补盲区**：研究并回答每个盲区，引用一手资料。
4. **第四步 — 精炼解释**：填补盲区后的改进版本。
5. **🧪 实践练习**：动手练习，检验真实理解。

---

## 核心参考资料

- [OpenAI — Harness engineering: leveraging Codex in an agent-first world](https://openai.com/index/harness-engineering/)
- [Anthropic — Effective harnesses for long-running agents](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents)
- [Anthropic — Building effective agents](https://www.anthropic.com/engineering/building-effective-agents)
- [HumanLayer — Skill Issue: Harness Engineering for Coding Agents](https://www.humanlayer.dev/blog/skill-issue-harness-engineering-for-coding-agents)
- [LangChain — The Anatomy of an Agent Harness](https://blog.langchain.com/the-anatomy-of-an-agent-harness/)
- [LangChain — Improving Deep Agents with harness engineering](https://blog.langchain.com/improving-deep-agents-with-harness-engineering/)
- [Thoughtworks — Harness Engineering](https://martinfowler.com/articles/exploring-gen-ai/harness-engineering.html)
- [HumanLayer — 12 Factor Agents](https://www.humanlayer.dev/blog/12-factor-agents)
- [Manus — Context Engineering for AI Agents](https://manus.im/blog/Context-Engineering-for-AI-Agents-Lessons-from-Building-Manus)
