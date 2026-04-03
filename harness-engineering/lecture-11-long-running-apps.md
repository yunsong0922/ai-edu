# 第11讲：长任务应用的 Harness 设计

> **费曼核心概念**：如何构建能在多个 context window、多小时、多个故障点中存活的 harness——用于单个 Agent 会话根本无法承载的任务。

---

## 第一步：简单解释

大多数 Agent 演示展示的是能在一个会话中完成的任务："编写这个函数"、"修复这个 bug"、"摘要这个文档"。

真实的软件项目不是这样运作的。构建一个完整的应用——甚至是一个重要的功能——可能需要数百次工具调用、涉及数十个文件，并跨越多天。没有任何 context window 能容纳这些。

朴素的做法：一直运行直到 context 满了，然后重新开始，希望模型能记住它做了什么。

工程化的做法：**设计 harness，使任务可以被暂停、恢复和干净地交接**——不丢失任何进度，不偏离原始目标。

### 类比 🏗️

摩天楼建设：没有任何一个工作班次能建完一栋摩天楼。每个班次以**现场报告**开始（完成了什么、正在进行什么、接下来是什么），在明确划定的区域内工作，并以**交接文档**结束，供下一个班次使用。Harness 就是使这种多班次工作连贯的项目管理系统。

---

## 第二步：识别盲区

| 盲区 | 我的说法 | 我不确定的地方 |
|------|---------|--------------|
| 1 | "暂停、恢复、干净地交接" | Anthropic 使用的具体机制是什么？ |
| 2 | "init.sh" | 这是什么，它做什么？ |
| 3 | "功能列表（feature list）" | Harness 如何使用功能列表来管理长任务？ |
| 4 | "自我验证（self-verification）" | Agent 在交接前如何验证自己的工作？ |

---

## 第三步：填补盲区

### 盲区1：长时间运行 Agent 的具体机制

**解答**（[Anthropic — Effective Harnesses for Long-Running Agents](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents)）：

Anthropic 的多 context window 设计使用三个相互锁定的机制：

**1. 初始化 Agent（Initializer Agent）**
一个在每个 context window 开始时运行的独立、轻量级 Agent。它的职责：
- 读取当前任务状态制品
- 为新 window 建立 context（完成了什么、接下来是什么）
- 初始化工作环境（加载正确的文件，配置工具）
- 交接给主实现 Agent

初始化 Agent 防止每个 window"冷启动"——它引导连续性。

**2. 交接制品（Handoff Artifacts）**
在每个 context window 结束时，Agent 写出一个结构化文档：
```markdown
# 任务交接：Auth 模块重构
## 已完成
- 提取了 UserValidator 类（src/auth/UserValidator.ts）
- 更新了12个调用点（参见：task/completed-sites.txt）
- 测试通过：auth.test.ts, validator.test.ts

## 进行中
- 调用点更新：src/api/routes/orders.ts（第47行）
- 已部分更新，需要：将 validateUser() 替换为 UserValidator.validate()

## 后续步骤
1. 完成 orders.ts 更新
2. 更新 src/api/routes/payments.ts（类似模式）
3. 运行完整测试套件

## 仍然有效的约束
- 不要修改 legacy/ 目录
- 所有更改必须保持与 v1 API 的向后兼容性

## 已修改的文件
[迄今为止所有已接触文件的列表]
```

**3. 编排者（Orchestrator）**
管理多 window 生命周期的外部编排者：
- 当当前 window 耗尽时触发新的 context window
- 将交接制品传递给初始化 Agent
- 监控整体进度
- 检测 Agent 是否在循环或卡住
- 处理升级到人工审查

---

### 盲区2：init.sh

**问题**：在长时间运行 Agent harness 的 context 中，`init.sh` 是什么？

**解答**（Anthropic 的 harness engineering 文章）：

`init.sh` 是 harness 在 Agent 开始工作之前运行的 shell 脚本。它将环境准备到已知状态：

```bash
#!/bin/bash
# init.sh — 在每次 agent 会话前运行

# 确保干净的工作状态
git status --porcelain
if [ ! -z "$(git status --porcelain)" ]; then
  echo "WARNING: 检测到未提交的更改"
fi

# 安装依赖（幂等）
npm install --silent

# 运行基线测试以建立起始状态
npm test --silent 2>&1 | tail -5

# 验证所需文件存在
[ -f "TASK_STATUS.md" ] || echo "未找到任务状态——从头开始"
[ -f "CLAUDE.md" ] || { echo "ERROR: CLAUDE.md 缺失"; exit 1; }

# 报告环境状态
echo "环境就绪。Node: $(node --version)"
echo "测试通过：$(npm test 2>&1 | grep 'Tests:' | tail -1)"
```

这意味着每次 Agent 会话都以以下内容开始：
- 已知的依赖状态
- 基线测试结果（这样 Agent 就知道什么是已经失败的）
- 已验证的所需文件
- 没有意外的环境差异

**简单版本**：`init.sh` 是起飞前检查清单。在每次会话前运行它，保证已知的起始状态。

---

### 盲区3：功能列表作为任务状态

**问题**：Harness 如何使用功能列表来管理长任务？

**解答**（Anthropic 长时间运行 harness 文章）：

不在 Agent 的 context 中追踪进度（这会丢失），harness 维护一个**外部功能列表（external feature list）**——仓库中持久追踪任务状态的文件：

```markdown
# feature-list.md

## 状态说明
- [ ] 未开始
- [~] 进行中
- [x] 完成
- [!] 阻塞——需要人工审查

## Auth 模块重构

### 第1阶段：提取 Validator
- [x] 创建 UserValidator 类
- [x] 编写单元测试
- [x] 更新 auth 路由

### 第2阶段：更新调用点
- [x] src/api/routes/auth.ts
- [~] src/api/routes/orders.ts（进行中）
- [ ] src/api/routes/payments.ts
- [ ] src/api/routes/admin.ts
- [!] src/legacy/old-auth.ts（阻塞：禁止修改）

### 第3阶段：清理
- [ ] 移除已废弃的 validateUser() 函数
- [ ] 更新 API 文档
- [ ] 运行完整回归测试套件
```

Agent 在会话开始时（通过初始化 Agent）读取此文件，工作时更新它，并在交接时将其更新后留下。编排者读取它以了解整体进度。

**简单版本**：功能列表是跨 context window 持久存在的共享任务板。这就是整个多 window 任务保持连贯的方式。

---

### 盲区4：交接前的自我验证

**问题**：Agent 在结束其 context window 之前如何验证自己的工作？

**解答**（Anthropic harness 设计 + [Thoughtworks — Assessing internal quality](https://martinfowler.com/articles/exploring-gen-ai/ccmenu-quality.html)）：

在写交接制品之前，Agent 运行自己的验证清单：

```
自我验证协议（在每次交接前运行）：

1. 测试：
   - 运行：npm test
   - 检查：与基线相比没有新的失败
   - 如果有新的失败：在交接前修复，或标记为 [!] 阻塞

2. 范围检查：
   - 列出所有已修改的文件
   - 验证没有触碰到分配范围之外的文件
   - 验证没有触碰 .env 文件、legacy/ 文件等

3. 一致性检查：
   - 是否有半完成的更改？（部分重构、注释掉的代码）
   - 如果有：完成或回滚——交接中不留半完成的工作

4. 功能列表更新：
   - 将已完成的项目标记为 [x]
   - 将进行中的项目标记为 [~] 并注明当前状态
   - 将新发现的阻塞项记录为 [!]

5. 写交接制品
```

**简单版本**：Agent 是自己的质量关卡。在交接之前，它运行人类会运行的检查。

---

## 第四步：精炼解释

### 长任务 Harness 架构

```
会话1                      会话2                      会话3
┌─────────────────┐          ┌─────────────────┐          ┌─────────────────┐
│  init.sh        │          │  init.sh        │          │  init.sh        │
│  读取任务状态    │          │  读取交接内容    │          │  读取交接内容    │
│  ↓              │          │  ↓              │          │  ↓              │
│  Agent 工作      │          │  Agent 工作      │          │  Agent 工作      │
│  ↓              │          │  ↓              │          │  ↓              │
│  自我验证        │          │  自我验证        │          │  自我验证        │
│  ↓              │ ──────→  │  ↓              │ ──────→  │  ↓              │
│  写交接制品      │ 交接制品  │  写交接制品      │ 交接制品  │  写交接制品      │
└─────────────────┘          └─────────────────┘          └─────────────────┘
         ↑ ↑                          ↑ ↑                          ↑ ↑
         │ └── feature-list.md ───────┘ └── feature-list.md ───────┘
         └──── CLAUDE.md（始终存在，永不修改）
```

### 每个制品中包含什么

| 制品 | 由谁更新 | 由谁读取 | 包含内容 |
|------|---------|---------|---------|
| `CLAUDE.md` | 人类（极少） | 每次会话 | 永久代码库规则 |
| `feature-list.md` | Agent（每轮） | 每次会话，编排者 | 当前任务状态 |
| `TASK_HANDOFF.md` | Agent（会话结束） | 下次会话的初始化 Agent | 完成了什么，接下来是什么 |
| `init.sh` | 人类（设置时） | Harness（每次会话前） | 环境准备命令 |

### 核心要点

1. 长任务需要外部状态——context window 无法在会话间存活。
2. 初始化 Agent 从已知状态引导每个新会话。
3. 功能列表 + 交接制品是跨 context window 的通信渠道。
4. `init.sh` 在每次会话前保证干净的环境——没有意外的状态差异。
5. 交接前的自我验证是防止不良状态传播的质量关卡。

---

### 30秒电梯演讲

> "长任务需要外部脚手架（scaffolding）：功能列表跨会话追踪状态，交接制品为下一个 window 做简报，init.sh 保证干净的起始，自我验证防止不良状态传播。"

---

## 🧪 实践 11

**系统设计练习**：为以下任务设计一个长任务 harness：
*"将代码库从 JavaScript 迁移到 TypeScript——150个文件，预计4-6小时工作量。"*

**A部分——init.sh**：为这个迁移任务编写 `init.sh`。在每次会话前应该检查和报告什么？

**B部分——功能列表**：设计 `feature-list.md` 结构。如何跨阶段组织150个文件？需要哪些状态？

**C部分——交接制品模板**：为 `TASK_HANDOFF.md` 编写 Markdown 模板——需要哪些章节，什么必须始终存在？

**D部分——自我验证**：Agent 在写交接前必须运行的5项检查是什么？要具体到 TypeScript 迁移（提示：类型错误、测试回归、意外遗留为 .js 的文件等）

**E部分——故障场景**：第3次会话结束时有3个 Agent 无法修复的 TypeScript 错误。交接如何处理这种情况？编排者怎么做？

---

## 延伸阅读

- [Effective harnesses for long-running agents](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents) — Anthropic 关于初始化 Agent、功能列表、init.sh 的核心文章
- [Harness design for long-running application development](https://www.anthropic.com/engineering/harness-design-long-running-apps) — Anthropic 关于任务状态和评估器设计的后续文章
- [Assessing internal quality while coding with an agent](https://martinfowler.com/articles/exploring-gen-ai/ccmenu-quality.html) — 将质量检查移入循环
- [EvoClaw benchmark](https://openhands.dev/blog/evoclaw-benchmark) — 跨真实仓库历史中依赖里程碑序列评估 Agent

---

*← [第10讲：Context Engineering 进阶](./lecture-10-context-engineering-advanced.md)*
*→ [第12讲：生产级 Harness 综合设计](./lecture-12-production-harness-capstone.md)*
