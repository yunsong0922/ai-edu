# 第12讲：生产级 Harness 综合设计——整合所有内容

> **费曼核心概念**：所有11讲如何组合成一个连贯的、生产级的 harness 设计——以及什么区分了在演示中有效的系统和在生产中有效的系统。

---

## 第一步：简单解释

我们已经覆盖了11个不同的支柱。每一个单独来看都很重要。但 harness engineering 的真正技能不在于了解每个支柱——而在于知道它们如何相互作用、在哪里产生冲突，以及如何做出权衡。

生产级 harness 不是一份检查清单。它是一个每个组件都依赖于并约束其他每个组件的系统。让其中一个出错，其他的就无法补偿。

本讲将一切综合成一个单一的连贯框架——并展示"生产级"实际上意味着什么。

### 类比 🏭

工厂流水线：每个工站（焊接、喷漆、装配、质量控制）单独工作都很好。但工厂只有在工站正确排序、吞吐量合适、异常处理一致，并且被设计来暴露——而不是隐藏——故障时才能正常运作。

---

## 第二步：十二要素 Harness（12-Factor Harness）

*综合了 HumanLayer 的 [12-Factor Agents](https://www.humanlayer.dev/blog/12-factor-agents) 和 [12-Factor AgentOps](https://www.12factoragentops.com/)，应用于所有11讲。*

| 要素 | 原则 | 对应讲次 |
|------|------|---------|
| **I. 单一职责（Single Responsibility）** | 每个组件只做一件事 | L1, L8, L9 |
| **II. 显式指令（Explicit Instructions）** | 所有 Agent 指导是可见且可审计的 | L4 |
| **III. Context 作为资源（Context as Resource）** | Context 被刻意预算，而非随意堆积 | L2, L10 |
| **IV. 无状态 Agent（Stateless Agents）** | Agent 是无状态的；harness 拥有状态 | L7, L11 |
| **V. 渐进式自主（Graduated Autonomy）** | 动作按风险分级；高风险需要人工关卡 | L3 |
| **VI. 最小足迹（Minimum Footprint）** | Agent 只申请当前任务所需的内容 | L3 |
| **VII. 显式工具（Explicit Tools）** | 工具可检查、最少且类型明确 | L9 |
| **VIII. 已验证的输出（Verified Output）** | Agent 在宣告完成前自我验证 | L5, L11 |
| **IX. 可观测（Observable）** | 每个动作都被记录、可追踪，并归因于 harness 版本 | L5, L7 |
| **X. 可恢复（Resumable）** | 任务可以被中断并在不丢失进度的情况下恢复 | L7, L11 |
| **XI. 可衡量（Measurable）** | 质量通过 evals 追踪，而不是直觉 | L5, L6 |
| **XII. 可演进（Evolvable）** | Harness 基于证据而非猜测而改进 | L5, L6 |

---

## 第三步：区分演示与生产的东西

### 演示/生产差距

| 维度 | 演示 | 生产 |
|------|------|------|
| 任务长度 | 单一会话 | 多会话、多天 |
| 错误处理 | 崩溃是可接受的 | 错误必须被恢复，不仅仅是记录 |
| 基础设施 | 开发者的笔记本 | 沙盒化、隔离、资源受限 |
| 可观测性 | Print 语句 | 带有 harness 版本的结构化追踪 |
| Evals | "看起来是对的" | 自动化、确定性、版本追踪 |
| 状态管理 | 隐式（在 context 中） | 显式（外部制品） |
| 并发 | 一次一个 Agent | 多个 Agent，并行工作流 |
| 人工监督 | 观察每一步 | 仅在风险边界处设检查点 |
| 恢复 | 从头重新开始 | 从最后一个检查点恢复 |
| 改进 | "尝试不同的 prompts" | 带有指标的受控 harness 实验 |

差距不在于模型质量。而在于每个非模型组件周围的工程纪律。

---

## 第四步：参考生产级 Harness

### 代码仓库结构

```
my-project/
├── CLAUDE.md                  ← 永久代码库规则（L4）
├── init.sh                    ← 环境准备（L11）
├── feature-list.md            ← 当前任务状态（L11）
│
├── .harness/
│   ├── tools/                 ← 工具定义和 schema（L9）
│   ├── evals/                 ← Eval 套件（L5, L6）
│   ├── prompts/               ← 系统 prompts（有版本）（L4）
│   └── traces/                ← 已记录的 Agent 追踪（L5, L7）
│
└── src/                       ← 你的实际代码
```

### Agent 循环（生产级）

```python
# 生产级 agent 循环的伪代码

def run_agent_session(task: Task, harness_version: str):
    # 1. 环境准备（L11）
    run_init_sh()
    
    # 2. 加载持久化状态（L11）
    task_state = read_feature_list()
    handoff = read_last_handoff()
    
    # 3. 初始化 context（L2, L10）
    context = ContextManager(
        budget_tokens=200_000,
        always_keep=["task_goal", "constraints", "current_phase"],
        compress_at=0.75,
        evict_policy="lru_except_always_keep"
    )
    context.load_system_prompt()         # KV 缓存位置（L10）
    context.load_agent_file("CLAUDE.md") # KV 缓存位置（L10）
    context.load_task_state(task_state)
    context.load_handoff(handoff)
    
    # 4. 为当前阶段配置工具（L9, L10）
    phase = task_state.current_phase
    tools = ToolRegistry.for_phase(phase)  # 工具屏蔽（L10）
    
    # 5. 运行 agent 循环
    trace = []
    while not task_state.complete:
        response = model.call(
            context=context.current(),
            tools=tools.available()
        )
        
        # 记录每个动作（L5）
        trace.append({
            "turn": len(trace),
            "response": response,
            "harness_version": harness_version
        })
        
        for tool_call in response.tool_calls:
            # 授权检查（L3）
            if not tools.is_authorized(tool_call, phase):
                raise HarnessViolation(f"未授权：{tool_call.name} 在 {phase}")
            
            # 如果需要，在沙盒中执行（L3, L9）
            result = tools.execute_sandboxed(tool_call)
            
            # 背压：在添加到 context 前过滤输出（L10）
            filtered_result = context.filter_tool_output(result)
            context.add_tool_result(filtered_result)
            
            # 更新外部状态（L11）
            task_state.update_from_tool(tool_call, result)
        
        # 如果需要压缩 context（L2, L10）
        if context.usage_ratio > 0.75:
            context.compress(strategy="progressive_summarization")
    
    # 6. 交接前自我验证（L5, L11）
    verification = run_self_verification_checklist()
    if not verification.passed:
        # 不交接损坏的状态
        raise VerificationFailed(verification.failures)
    
    # 7. 写交接制品（L11）
    write_handoff(task_state, trace_summary=summarize(trace))
    
    # 8. 存储追踪用于 evals（L5, L6）
    eval_store.save(trace, harness_version=harness_version)
    
    return SessionResult(success=True, trace=trace)
```

---

## 第五步：Harness 成熟度模型

用这个来评估你在哪里，以及接下来要改进什么。

### 第0级——无 Harness
- 直接调用模型 API
- 没有持久化状态，没有指令
- *结果*：不一致，不可靠，不可重现

### 第1级——基础指令
- 带有代码库 context 的 `CLAUDE.md`
- 基础工具集
- *结果*：更加一致，但没有系统性的故障恢复

### 第2级——Context 管理
- 主动 context 压缩
- 多会话任务的交接制品
- *结果*：长任务变得可行

### 第3级——已评估
- 带有确定性验证器的 eval 套件
- 带有 harness 版本的追踪日志
- 交接前的自我验证
- *结果*：可以衡量质量，可以检测回归

### 第4级——受控自主
- 按阶段的工具权限矩阵
- 沙盒化的代码执行
- Prompt 注入防御
- 人工检查点触发器
- *结果*：对具有真实副作用的任务足够安全

### 第5级——生产级
- 以上所有内容
- 持久化执行（崩溃恢复、恢复）
- 带有结构化交接的多 Agent 协调
- 可重现环境的 `init.sh`
- 基于基准测试的 harness 迭代
- *结果*：在生产规模下可靠，可基于证据改进

2026年，大多数团队处于第1-2级。第3级将严肃工程与"感觉派编码"区分开来。第5级是前沿。

---

## 最终综合练习

为以下场景设计一个**完整的生产级 harness**：

> **场景**：你的团队想使用 AI Agent 来处理传入的 GitHub issue。对于每个新 issue：
> 1. 分类（分配标签、优先级、分配给正确的团队）
> 2. 如果是 bug：尝试复现并产出一个最小复现用例
> 3. 如果是功能请求：起草一份初始设计文档
> 4. 在 issue 上发布第一条回复
>
> 这在真实的生产代码仓库上全天候（24/7）持续运行。

对于每个讲次主题，写出你的 harness 设计：

| 主题 | 你的设计 |
|------|---------|
| **L1 — 什么是 harness？** | 所有 harness 组件是什么？ |
| **L2 — Context 与记忆** | 什么在 context 中？什么被摘要？交接是什么？ |
| **L3 — 约束** | 它可以自主做什么？什么需要人工批准？什么是禁止的？ |
| **L4 — Specs 与 Agent Files** | CLAUDE.md 中放什么？每个 issue 得到什么 spec？ |
| **L5 — Evals** | 你怎么知道一次分类是正确的？你的 eval 套件是什么？ |
| **L6 — Benchmarks** | 哪个公共基准测试最接近这个任务？ |
| **L7 — Runtime** | 你需要什么 runtime 基础设施？ |
| **L8 — 多 Agent** | 你需要多个 Agent 吗？有什么角色？ |
| **L9 — 工具设计** | 列出5-7个工具及其签名 |
| **L10 — 进阶 Context** | 你应用什么背压 + 工具屏蔽？ |
| **L11 — 长任务** | init.sh 做什么？功能列表里有什么？ |
| **L12 — 成熟度级别** | 你的设计达到什么成熟度级别？第5级需要什么？ |

---

## 结语洞见

本课程中的每一讲都可以用一句话总结：

> **模型是一个组件。Harness 是其他一切。Harness 决定了模型是否有用。**

更好的 harness 胜过更好的模型——持续地、可衡量地、可重现地。研究是明确的（[LangChain](https://blog.langchain.com/improving-deep-agents-with-harness-engineering/)、[HumanLayer](https://www.humanlayer.dev/blog/skill-issue-harness-engineering-for-coding-agents)、[Anthropic](https://www.anthropic.com/engineering/infrastructure-noise)）：harness 质量主导结果质量。

这是 harness engineering 作为一门学科的核心赌注：**瓶颈不是模型。而是它周围的环境。**

---

## 完整课程参考

| 讲次 | 主题 | 核心洞见 |
|------|------|---------|
| 1 | 什么是 Harness？ | 模型是一个组件；harness 是其他一切 |
| 2 | Context 与记忆 | Context window = RAM；刻意管理它 |
| 3 | 约束与安全自主性 | 护栏（guardrails）让速度更快；prompt 注入需要 harness 防御 |
| 4 | Specs 与 Agent Files | 持久化指令；CLAUDE.md 是 Agent 的 onboarding 文档 |
| 5 | Evals 与可观测性 | 衡量 harness 质量，而不仅仅是模型质量 |
| 6 | Benchmarks | 相同模型，更好的 harness = 更好的分数；控制你的变量 |
| 7 | Runtimes | 持久化执行 + runtime 基础设施 = 生产可靠性 |
| 8 | 多 Agent 系统 | 结构化制品 + 角色分离 = 无冲突的协调 |
| 9 | 工具设计 | 一个工具，一个动作；糟糕的接口导致系统性 Agent 错误 |
| 10 | 进阶 Context 工程 | 主动压缩，屏蔽工具，施加背压 |
| 11 | 长任务应用 | init.sh + 功能列表 + 交接制品 = 跨会话连贯性 |
| 12 | 生产级综合设计 | 所有支柱整合；成熟度模型帮你定位现状 |

---

## 延伸阅读（综合）

- [Harness engineering: leveraging Codex in an agent-first world](https://openai.com/index/harness-engineering/) — 旗舰性的实地报告
- [Effective harnesses for long-running agents](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents) — Anthropic 的核心文章
- [Building effective agents](https://www.anthropic.com/engineering/building-effective-agents) — 更广泛的 Anthropic 指南
- [12 Factor Agents](https://www.humanlayer.dev/blog/12-factor-agents) — 操作原则综合
- [12-Factor AgentOps](https://www.12factoragentops.com/) — 面向运营的补充
- [Skill Issue: Harness Engineering for Coding Agents](https://www.humanlayer.dev/blog/skill-issue-harness-engineering-for-coding-agents) — 弱结果 = harness 问题
- [Improving Deep Agents with harness engineering](https://blog.langchain.com/improving-deep-agents-with-harness-engineering/) — harness 影响的实测证据
- [Harness Engineering (Thoughtworks)](https://martinfowler.com/articles/exploring-gen-ai/harness-engineering.html) — Context 工程、约束、熵
- [walkinglabs/learn-harness-engineering](https://github.com/walkinglabs/learn-harness-engineering) — 基于项目的课程仓库

---

*← [第11讲：长任务应用](./lecture-11-long-running-apps.md)*
*→ 返回[课程概览](./README.md)*
