# 第5讲：Evals 与可观测性——你怎么知道它起效了？

> **费曼核心概念**：如何衡量 Agent 是否真正做了好的工作，以及如何看透它的决策过程。

---

## 第一步：简单解释

你让 Agent 去修复10个 bug。它回来说"完成了！"

它真的修好了吗？在此过程中有没有破坏其他5件事？它是否走了一条不必要的昂贵路径——读了50个文件，其实5个就够了？它有没有给出一个能编译但逻辑上错误的幻觉式修复？

**你不知道——除非你在 harness 中内置了评估（evaluation）机制。**

**Evals（评估）**是对 Agent 行为的测试。不仅仅是"代码能不能编译"——而是：Agent 是否遵循了正确的轨迹？它是否使用了正确的工具？它是否正确处理了模糊情况？

**可观测性（Observability）**是记录并追踪 Agent 所做一切的手段，让你能理解它*为什么*成功或失败——并系统性地改进 harness。

### 类比 🏥

医生做体检：你不会只问"你感觉还好吗？"你会做检查——血压、胆固醇、反射。每项检查针对一种特定的故障模式。Agent evals 也一样——每个 eval 针对 harness 中一种特定的可靠性故障模式。

---

## 第二步：识别盲区

| 盲区 | 我的说法 | 我不确定的地方 |
|------|---------|--------------|
| 1 | "Evals 是对 Agent 行为的测试" | Agent evals 与普通单元测试有什么区别？ |
| 2 | "它是否遵循了正确的轨迹？" | 什么是轨迹级别（trajectory-level）评估？ |
| 3 | "基础设施噪声（infrastructure noise）" | 这是什么意思，为什么我应该关心？ |
| 4 | "可观测性" | 对于一个 Agent，我应该具体记录什么？ |

---

## 第三步：填补盲区

### 盲区1：Agent Evals vs 单元测试

**问题**：我的代码库已经有单元测试了，为什么还需要 evals？

**解答**（[OpenAI — Testing Agent Skills Systematically](https://developers.openai.com/blog/eval-skills/) + [OpenHands — How to Evaluate Agent Skills](https://openhands.dev/blog/evaluating-agent-skills)）：

单元测试检查特定函数的*输出正确性*。  
Agent evals 检查 Agent 在完成任务时的*行为正确性*：

| 维度 | 单元测试 | Agent Eval |
|------|---------|-----------|
| 测试对象 | 单个函数的输出 | Agent 的任务完成情况 |
| 输入 | 确定性输入 | 真实的任务描述 |
| 成功标准 | 返回值符合预期 | 任务目标达成（可能有多条有效路径） |
| 不确定性 | 无 | 高（Agent 可以通过多种方式成功） |
| 揭示的问题 | 代码 bug | Harness 缺口、技能缺口、推理失败 |

Agent evals 需要**确定性验证器（deterministic verifiers）**——自动检查任务目标是否达成，与 Agent 采取的路径无关。

**简单版本**：单元测试说"这个函数返回42。"Evals 说"Agent 正确处理了模糊的用户请求，并产出了可运行的、经过测试的代码。"

---

### 盲区2：轨迹级别评估（Trajectory-Level Evaluation）

**问题**：评估*轨迹*而不仅仅是最终结果，意味着什么？

**解答**（[OpenAI — Trace Grading](https://platform.openai.com/docs/guides/trace-grading) + [LangChain — Evaluating Deep Agents](https://blog.langchain.com/evaluating-deep-agents-our-learnings/)）：

轨迹（trajectory）是 Agent 所采取动作的完整序列：调用了哪些工具、以什么顺序、带什么参数、对结果做了什么。

两个 Agent 可能都通过了最终测试（"代码能运行"），但轨迹截然不同：
- Agent A：读3个文件 → 写修复 → 运行测试 → 完成
- Agent B：读50个文件 → 重写8个文件 → 破坏2个测试 → 修复它们 → 完成

结果相同。但 Agent B 的成本是10倍，耗时5倍，并引入了不必要的大量改动。

轨迹级别评估能捕获：
- **低效**——用10次工具调用就能完成的事用了100次
- **错误推理**——通过无效逻辑得到了正确答案（边缘案例会失败）
- **利用 harness 的行为**——找到了不可泛化的意外捷径

LangChain 定义了三个评估级别：
1. **单步（Single-step）**——这次工具调用产出了正确结果吗？
2. **完整运行（Full-run）**——Agent 完成任务了吗？
3. **多轮（Multi-turn）**——Agent 在长会话中保持了连贯的行为吗？

**简单版本**：最终答案不是唯一重要的事情。Agent *如何*得到答案，揭示了它是真正理解了问题，还是只是碰巧运气好。

---

### 盲区3：基础设施噪声（Infrastructure Noise）

**问题**：Evals 中的"基础设施噪声"是什么，为什么重要？

**解答**（[Anthropic — Quantifying infrastructure noise](https://www.anthropic.com/engineering/infrastructure-noise)）：

Anthropic 发现，*仅运行时配置*——温度（temperature）设置、工具超时值、沙盒（sandbox）配置等——就能让基准测试分数的变化幅度，超过排行榜上许多模型层级之间的差异。

这意味着：如果你的 eval 环境与生产环境不同，你的 eval 结果就会产生误导。你可能测量的是 harness，而不是模型。或者更糟——测量的是环境干扰，而不是两者。

**简单版本**：如果你的 eval 在快速沙盒中运行，但生产环境在有超时的慢速网络上运行，你的 eval 分数就是虚构的。让 eval 环境与生产环境匹配。

---

### 盲区4：可观测性：记录什么

**问题**：我应该在 Agent harness 中具体检测什么？

**解答**（[Anthropic — Demystifying Evals for AI Agents](https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents) + [OpenHands — Learning to Verify AI-Generated Code](https://openhands.dev/blog/20260305-learning-to-verify-ai-generated-code)）：

一份 Agent 追踪（trace）应该捕获：

```
对 agent 循环中的每一轮（turn）：
  - timestamp（时间戳）
  - input（Agent 收到的内容）
  - reasoning（如果可用，推理过程）
  - tool_calls: [{name, params, result, latency_ms}]
  - output（Agent 产出的内容）
  - token_count（输入 + 输出）
  - cost_usd（成本）

对整个任务：
  - task_id
  - task_description
  - success: boolean
  - success_criteria_met: [检查项列表]
  - total_turns
  - total_tokens
  - total_cost_usd
  - wall_clock_time_s
  - harness_version（！重要）
```

`harness_version` 字段至关重要——当你改进 harness 时，你需要知道哪些追踪是由哪个版本产生的。

**简单版本**：记录 Agent 做的一切、花费了多少，以及是否成功。用 harness 版本标记每条追踪，这样你就能衡量随时间的改进。

---

## 第四步：精炼解释

### 每个 Eval 必须回答的四个问题

| 问题 | Eval 类型 | 示例检查 |
|------|---------|---------|
| 它完成任务了吗？ | 结果（Outcome） | 测试通过了吗？功能正常工作了吗？ |
| 它正确完成了吗？ | 正确性（Correctness） | 有没有回归？逻辑是否合理？ |
| 它高效完成了吗？ | 轨迹（Trajectory） | 调用了多少次工具？消耗了多少 tokens？ |
| 在更难的情况下还能工作吗？ | 泛化（Generalization） | 对任务的10个变体运行测试 |

### Eval 金字塔

```
              ▲
             / \
            / E \ ← E2E / Full-run evals（昂贵、慢，捕获 harness 级别故障）
           /─────\
          / Multi \ ← Multi-turn evals（中等成本，捕获漂移和连贯性故障）
         /─────────\
        / Unit/Step  \ ← Single-step evals（便宜、快，捕获工具调用正确性）
       /─────────────\
```

便宜的持续运行。昂贵的在重大 harness 变更时运行。

### 最小化 Eval 设置

```python
# Agent eval 的伪代码

def eval_harness(agent, task, expected_outcome):
    trace = []
    result = agent.run(
        task=task,
        on_tool_call=lambda call: trace.append(call)
    )
    
    checks = {
        "task_completed": verify_outcome(result, expected_outcome),
        "no_regressions": run_test_suite(),
        "tool_calls_within_budget": len(trace) < MAX_TOOL_CALLS,
        "token_cost_within_budget": sum(t.tokens for t in trace) < MAX_TOKENS,
    }
    
    return EvalResult(
        success=all(checks.values()),
        checks=checks,
        trace=trace,
        harness_version=HARNESS_VERSION
    )
```

### 核心要点

1. Evals 衡量 harness 质量，而不仅仅是模型质量。相同模型，更好的 harness = 更好的 eval 分数。
2. 轨迹评估能捕获结果评估遗漏的问题。
3. 让 eval 环境与生产环境匹配——基础设施噪声可能比模型差异更大。
4. 用 harness 版本标记每条追踪，这是衡量进步的方式。

---

### 30秒电梯演讲

> "Evals 回答：它起效了吗？在变好吗？没有 evals，你是在盲飞——你无法区分 harness 问题和模型问题，也无法衡量改进。"

---

## 🧪 实践 5

**设计练习**：为实践1中的 PR Review Agent 设计一套 eval 套件。

**A部分——结果 Eval（Outcome Eval）**：为"Agent 正确审查了这个 PR"写出3个具体的、可验证的成功标准。  
（提示：每个标准必须可以通过脚本检查，而不仅仅是人类判断。）

**B部分——轨迹 Eval（Trajectory Eval）**：即使结果正确，什么样的轨迹算是"坏轨迹"？  
（提示：想想工具调用次数、它读取了哪些文件、token 成本。）

**C部分——Eval 数据集**：列出你要在 eval 集中包含的5个不同的 PR 场景，使其更加健壮。  
（提示：考虑边缘案例——空的 PR、巨大的 PR、破坏测试的 PR、涉及敏感文件的 PR。）

**D部分——基础设施匹配**：你的 Agent 在 GitHub Actions 沙盒中运行 evals。列出2种 eval 环境可能与生产环境不同、从而使 eval 分数失效的方式。

---

## 综合练习：整合所有内容

你已经覆盖了所有5个支柱。为以下场景设计一个**完整的 harness**：

> **任务**：构建一个 Agent，监控你的 GitHub 仓库中的新 issue，对其进行分类（分配标签、优先级），并起草第一条回复。

对于每个支柱，写2-3句话：

| 支柱 | 你的 Harness 设计 |
|------|-----------------|
| **第1讲：什么是 harness？** | 这个 harness 由哪些组件组成？ |
| **第2讲：Context 与记忆** | 什么保留在 context 中？什么被摘要？交接制品（handoff artifact）是什么？ |
| **第3讲：约束与安全自主性** | 它可以自主做什么？什么需要检查点？什么是被禁止的？ |
| **第4讲：Specs 与 Agent Files** | `CLAUDE.md` 中放什么？每个 issue 获得什么 spec？ |
| **第5讲：Evals 与可观测性** | 你怎么知道一次分类是正确的？你记录什么？ |

---

## 延伸阅读

- [Testing Agent Skills Systematically with Evals](https://developers.openai.com/blog/eval-skills/) — OpenAI：将追踪转化为可重复的 evals
- [How to Evaluate Agent Skills](https://openhands.dev/blog/evaluating-agent-skills) — OpenHands：有界任务、确定性验证器、追踪审查
- [Demystifying Evals for AI Agents](https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents) — Anthropic：当 Agent 有多条成功路径时，衡量什么
- [Quantifying infrastructure noise in agentic coding evals](https://www.anthropic.com/engineering/infrastructure-noise) — Anthropic：eval 环境很重要
- [Evaluating Deep Agents: Our Learnings](https://blog.langchain.com/evaluating-deep-agents-our-learnings/) — LangChain：单步、完整运行和多轮 eval 设计
- [Improving Deep Agents with harness engineering](https://blog.langchain.com/improving-deep-agents-with-harness-engineering/) — LangChain：证据表明 harness 改变能提升基准成绩
- [Trace grading](https://platform.openai.com/docs/guides/trace-grading) — OpenAI：直接对 Agent 追踪打分
- [Learning to Verify AI-Generated Code](https://openhands.dev/blog/20260305-learning-to-verify-ai-generated-code) — OpenHands：使用轨迹批评者进行分层验证

---

*← [第4讲：Specs、Agent Files 与工作流设计](./lecture-04-specs-agent-files-workflow.md)*
*→ 返回[课程概览](./README.md)*
