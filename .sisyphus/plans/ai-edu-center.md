# 本妙坊未来成长中心 — AI创造课程规划

## TL;DR

> **核心目标**: 为本妙坊未来成长中心设计完整的"AI创造课"独立科目体系，让零编程基础的6-15岁学生在约10个月内达到L4水平（独立从想法到完整应用）。
> 
> **交付物**:
> - 课程哲学与能力等级定义文档
> - 四阶段课程大纲（对话者→创造者→建造者→设计者），每阶段含4+课时计划
> - 基础设施与成本模型
> - 教师指南与培训方案
> - 评估体系与作品集模板
> - 20+挑战库（含年龄分层）
> - 全套文档集成审查
> 
> **预估工作量**: Large
> **并行执行**: YES — 4 waves
> **关键路径**: 课程哲学 → 四阶段课程 → 挑战库 → 集成审查

---

## Context

### 原始需求
赵老师（大宝的围棋老师）创办了"本妙坊未来成长中心"——一所替代性全日制学校。中心现有科目包括围棋、哲思、运动、辩论、数学。希望新增"AI创造课"作为独立科目，让零基础的小学生和中学生（混合教学）最终能使用 ClaudeCode/OpenCode 等AI工具，独立设计并开发完整应用。

### 讨论摘要

**已确认的决策**:
- **课程定位**: 独立科目（"AI创造课"），与围棋、数学等并列
- **目标水平**: L4（从想法到完整可用应用，含设计思考），循序渐进 L1→L4
- **教学本质**: 培养"小产品经理 + AI驾驶员"，非传统编程教学
- **工具选择**: 国外AI工具为主（Claude/ClaudeCode/OpenCode），启蒙阶段灵活
- **年龄差异处理**: 混合协作，中学生当"小老师"（大带小）
- **预算**: 有弹性，愿意为好工具付费
- **挑战设计**: 挑战驱动学习，但难度适中

**关键约束**:
- 学生完全零编程基础（10-20人中班，小学生与中学生各半）
- 教学老师可能没有编程背景
- 一年期课程已运行1个月（到2025年12月底）
- 中心提供电脑
- 全日制学校，课程灵活度高

**核心洞察**:
- 围棋训练的战略拆解能力、辩论训练的表达能力、哲思课的批判思维——这些正是AI编程最需要的底层能力
- 老师不需要会编程，需要会引导学生跟AI对话
- AI工具进化很快，课程设计应基于"能力"而非"特定工具界面"

### Metis 审查

**识别的关键缺口（已纳入方案）**:
- 学生数字素养基线评估（会打字吗？会用浏览器吗？）
- L4 定义需要按年龄分层（7岁的L4 ≠ 15岁的L4）
- 教师自身需要先体验AI工具（至少2周）
- 未成年人使用云AI服务的数据隐私合规
- 家长沟通策略（替代性学校家长对屏幕时间可能敏感）
- 工具/网络中断的应急方案
- 各阶段需要离线备用活动
- 学生项目存储与版本管理方案
- API 费用失控的防护机制

**后续补充的关键设计决策**:
- **Debug 困境解决方案**（组合策略）：
  1. **标准化求助流程**：截图→描述现象→发给AI→按建议操作。教师只需引导学生走流程，不需要懂技术
  2. **远程兼职 TA**：配备1-2名有编程经验的兼职技术支援，学生卡住时可呼叫"会诊"
  3. **渐进式代码素养**：从阶段二起逐步培养"读代码"能力——不教写代码，但教看懂错误信息、理解项目大致结构
- 这将原方案的"完全不碰代码"调整为"不写代码，但逐步学会读代码"

---

## Work Objectives

### 核心目标
设计一套完整的、可执行的"AI创造课"课程体系及配套支撑方案，使非技术背景的教师能够独立教学，带领零基础的6-15岁学生在约10个月内达到AI辅助应用开发的L4水平。

### 具体交付物
1. 课程哲学与L1-L4能力等级定义文档（含年龄分层）
2. 阶段一课程：对话者（Dialoguer）— 完整课时计划
3. 阶段二课程：创造者（Creator）— 完整课时计划
4. 阶段三课程：建造者（Builder）— 完整课时计划
5. 阶段四课程：设计者（Designer）— 完整课时计划
6. 基础设施与成本模型
7. 教师指南与培训方案
8. 评估体系与作品集模板
9. 挑战库（20+挑战，跨阶段，含年龄分层）
10. 全套文档集成审查与快速上手指南

### 完成标准
- [ ] 所有课时计划通过"非技术教师可执行"测试
- [ ] 每个挑战/活动有小学版和中学版两个变体
- [ ] 成本模型基于真实API定价，可验证
- [ ] 每个阶段有明确的作品集转段检查点
- [ ] 全套文档无内部矛盾

### Must Have
- 每堂课必须有可见的学生产出物（不能只"学概念"不"造东西"）
- 每个阶段有离线备用方案（网络/API中断时怎么办）
- 教师指南不使用任何代码术语
- 所有学生面向材料基于中文设计
- 挑战难度标定：80%学生应能完成（低于80%说明太难）

### Must NOT Have（护栏）
- ❌ 传统编程教学（不教写代码语法，但阶段二起逐步培养"读代码"能力）
- ❌ 需要教师理解代码才能执行的课时计划（教师依赖标准化流程和TA支援）
- ❌ 自建 LMS/教学平台（用现有工具如 Notion/Google Docs/纸质作品集）
- ❌ 独立的"提示词工程理论"模块（提示能力在实践中习得）
- ❌ 学术化的AI伦理课程（伦理融入项目实践，不单独开课）
- ❌ 针对单一年龄段的设计（所有内容必须有年龄分层）
- ❌ 每阶段引入超过2个新工具（避免工具过载）
- ❌ 47维评估矩阵（用简单3级评估：未达到/进步中/已掌握）

---

## Verification Strategy

> **全部验证由执行 Agent 完成** — 无需人工干预。

### 文档质量验证
- **非技术教师测试**: 每份课时计划交给一个不懂代码的人阅读，确认他们能理解如何执行
- **年龄分层检查**: 每个活动/挑战必须有≥2个年龄变体
- **成本验证**: 所有费用估算对照 Claude API 官方定价页面交叉验证
- **内部一致性**: 全套文档交叉引用，无矛盾

### QA 策略
每个任务完成后，执行 Agent 需：
1. 重新阅读任务的验收标准
2. 逐条检查并记录通过/未通过
3. 任何未通过项必须修复后才能标记完成

---

## Execution Strategy

### 并行执行波次

```
Wave 1 (立即开始 — 基础文档 + 独立支撑文档):
├── Task 1: 课程哲学与L级定义 [writing] ★关键路径
├── Task 6: 基础设施与成本模型 [deep]
├── Task 7: 教师指南与培训方案 [writing]
└── Task 8: 评估体系与作品集模板 [writing]

Wave 2 (Task 1 完成后 — 四阶段课程，最大并行):
├── Task 2: 阶段一课程：对话者 (depends: 1) [writing]
├── Task 3: 阶段二课程：创造者 (depends: 1) [writing]
├── Task 4: 阶段三课程：建造者 (depends: 1) [writing]
└── Task 5: 阶段四课程：设计者 (depends: 1) [writing]

Wave 3 (Tasks 2-5 完成后 — 挑战库):
└── Task 9: 挑战库 (depends: 2,3,4,5) [writing]

Wave FINAL (所有任务完成后 — 集成审查):
└── Task 10: 集成审查与快速上手指南 (depends: 1-9) [deep]
→ 呈现结果 → 获取用户确认

关键路径: Task 1 → Tasks 2-5 → Task 9 → Task 10
并行加速: ~60% 快于顺序执行
最大并发: 4 (Wave 1 & Wave 2)
```

### 依赖矩阵

| Task | 依赖 | 被依赖 | Wave |
|------|------|--------|------|
| 1. 课程哲学与L级定义 | 无 | 2,3,4,5,9 | 1 |
| 6. 基础设施与成本 | 无 | 10 | 1 |
| 7. 教师指南 | 无 | 10 | 1 |
| 8. 评估体系 | 无 | 10 | 1 |
| 2. 阶段一：对话者 | 1 | 9,10 | 2 |
| 3. 阶段二：创造者 | 1 | 9,10 | 2 |
| 4. 阶段三：建造者 | 1 | 9,10 | 2 |
| 5. 阶段四：设计者 | 1 | 9,10 | 2 |
| 9. 挑战库 | 2,3,4,5 | 10 | 3 |
| 10. 集成审查 | 1-9 | 无 | FINAL |

### Agent 调度摘要

- **Wave 1**: **4 并行** — T1 → `writing`, T6 → `deep`, T7 → `writing`, T8 → `writing`
- **Wave 2**: **4 并行** — T2-T5 → `writing`
- **Wave 3**: **1** — T9 → `writing`
- **Wave FINAL**: **1** — T10 → `deep`

---

## TODOs

- [ ] 1. 课程哲学与 L1-L4 能力等级定义

  **What to do**:
  - 撰写课程哲学声明（<500字）：为什么教"AI驾驶"而非"编程"，这所学校的独特教育理念
  - 定义四阶段模型（对话者→创造者→建造者→设计者），每阶段的入口条件和出口条件
  - 定义 L1-L4 能力等级，每个等级按年龄分两个变体：
    - 小学版（6-10岁）：更具象、更视觉化的标准
    - 中学版（11-15岁）：更抽象、更结构化的标准
  - 每个等级×年龄组提供≥2个具体作品示例（共≥16个示例）
  - 绘制跨课程能力迁移图：围棋→战略拆解、辩论→需求表达、哲思→伦理设计判断
  - 定义阶段转段标准：基于作品集证据，非基于时间

  **Must NOT do**:
  - 不要使用任何编程术语
  - 不要设计针对单一年龄段的定义
  - 不要把阶段转段设计为考试制

  **Recommended Agent Profile**:
  - **Category**: `writing`
    - Reason: 这是一份教育理念与能力框架文档，核心是清晰的文字表达
  - **Skills**: [`brainstorming`]
    - `brainstorming`: 需要创造性思考如何将围棋/辩论/哲思能力映射到AI编程能力

  **Parallelization**:
  - **Can Run In Parallel**: YES
  - **Parallel Group**: Wave 1 (with Tasks 6, 7, 8)
  - **Blocks**: Tasks 2, 3, 4, 5, 9（所有课程内容依赖此文档的L级定义）
  - **Blocked By**: None (can start immediately)

  **References**:
  - **Draft**: `.sisyphus/drafts/ai-edu-center.md` — 完整的讨论记录和决策
  - **类比参考**: 围棋的段位制（从入门到专业段位的渐进体系）可作为L级设计的灵感

  **Acceptance Criteria**:
  - [ ] 哲学声明<500字，非技术人员可理解
  - [ ] 4阶段描述，每阶段有明确的入口/出口条件
  - [ ] L1-L4定义，每级有2个年龄变体（共8个定义）
  - [ ] 每级×每年龄组≥2个作品示例（共≥16个示例）
  - [ ] 跨课程能力迁移图包含≥3门现有科目
  - [ ] 转段标准基于作品集证据

  **QA Scenarios**:

  ```
  Scenario: 非技术人员可读性测试
    Tool: Bash (人工模拟 — Agent 自检)
    Steps:
      1. 阅读整个文档，检查是否出现任何编程术语（function, variable, API, code, debug 等）
      2. 检查每个L级定义是否有具体可观测的学生行为描述
      3. 验证16个作品示例是否具体（不是"做一个应用"而是"做一个班级生日提醒网页"）
    Expected Result: 零编程术语；所有定义基于可观测行为；所有示例具体可感知
    Evidence: docs/01-philosophy-and-levels.md

  Scenario: 年龄分层完整性检查
    Tool: Bash (grep)
    Steps:
      1. 检查L1-L4每个等级是否都有"小学版"和"中学版"两个变体
      2. 检查两个变体之间的难度差异是否合理（不是仅仅换个措辞）
    Expected Result: 8个等级定义（4级×2年龄组），变体间有实质性差异
    Evidence: docs/01-philosophy-and-levels.md
  ```

  **Commit**: YES
  - Message: `docs: 课程哲学与能力等级定义`
  - Files: `docs/01-philosophy-and-levels.md`

- [ ] 6. 基础设施与成本模型

  **What to do**:
  - 设计技术基础设施方案：
    - 电脑硬件要求（OS、配置、屏幕尺寸 — 需支持终端工具）
    - 网络要求（带宽、稳定性、翻墙/VPN 方案 — 访问国外AI服务必需）
    - AI工具账号管理方案（团队API vs 个人账号、未成年人使用管理）
  - 建立成本模型：
    - 每阶段的API使用量估算（基于：20学生 × 每周N小时上机 × 平均token消耗）
    - 工具订阅费用（Claude API、可能的备选工具）
    - **远程 TA 费用**（1-2名兼职技术支援的人力成本估算）
    - 硬件摊销
    - 月度总成本 / 生均成本
  - AI工具对比矩阵（≥3个工具）：Claude vs ChatGPT vs 其他，维度包括：成本、中文能力、代码能力、安全过滤、API稳定性
  - API 费用失控防护机制（速率限制、每日预算上限、token 监控）
  - 工具/网络中断应急方案（具体的离线活动清单）
  - 中国境内使用国外AI服务的网络方案与数据隐私考量
  - 学生项目存储方案（云端 vs 本地、备份策略）
  - 屏幕时间管理策略

  **Must NOT do**:
  - 不要推荐自建平台
  - 不要忽视网络翻墙的现实问题
  - 不要使用过时的API定价数据

  **Recommended Agent Profile**:
  - **Category**: `deep`
    - Reason: 需要深入研究真实定价、技术方案、合规要求，涉及多维度分析
  - **Skills**: []

  **Parallelization**:
  - **Can Run In Parallel**: YES
  - **Parallel Group**: Wave 1 (with Tasks 1, 7, 8)
  - **Blocks**: Task 10
  - **Blocked By**: None

  **References**:
  - **Claude API Pricing**: https://www.anthropic.com/pricing — 最新定价
  - **Draft**: `.sisyphus/drafts/ai-edu-center.md` — 预算讨论记录

  **Acceptance Criteria**:
  - [ ] 硬件规格表含最低/推荐两档
  - [ ] 月度成本分解到每个学生，含计算公式
  - [ ] 20人并发使用的API速率限制分析
  - [ ] 工具对比矩阵≥3个工具
  - [ ] 网络中断应急方案含具体离线活动
  - [ ] 中国特定的网络方案与数据隐私说明
  - [ ] API费用防护机制设计

  **QA Scenarios**:

  ```
  Scenario: 成本模型可验证性
    Tool: Bash (curl / web research)
    Steps:
      1. 访问 Claude API 定价页面，验证文档中引用的价格是否准确
      2. 检查token消耗估算的计算过程是否合理（输入假设 × 单价 = 总成本）
      3. 验证月度总成本是否与各项明细之和一致
    Expected Result: 价格数据与官方一致；计算逻辑透明可追溯
    Evidence: docs/06-infrastructure-and-cost.md

  Scenario: 应急方案实用性
    Tool: Bash (Agent 自检)
    Steps:
      1. 假设网络完全中断，检查应急方案是否提供了≥3个具体的离线活动
      2. 每个离线活动是否有明确的时间、材料、步骤
    Expected Result: ≥3个离线活动，每个都有完整描述
    Evidence: docs/06-infrastructure-and-cost.md
  ```

  **Commit**: YES
  - Message: `docs: 基础设施与成本模型`
  - Files: `docs/06-infrastructure-and-cost.md`

- [ ] 7. 教师指南与培训方案

  **What to do**:
  - 定义教师角色：引导者（facilitator），而非讲师（instructor）
  - 设计教师自我培训计划（2周）：
    - 教师自己先用AI工具完成一系列任务（从简单到复杂）
    - 每天的具体活动安排
    - 培训结束后的自我评估清单
  - 编写每堂课的教师准备清单模板（每课≤1页）
  - 编写常见学生问题FAQ（≥15条）：
    - "AI说的不对怎么办？"
    - "我不知道跟AI说什么"
    - "代码报错了"（教师不需要懂代码，但需要知道引导学生怎么跟AI描述错误）
  - 编写"不需要懂代码"安心指南——解释为什么不懂代码反而是优势
  - **设计标准化 Debug 求助流程**（核心新增）：
    - 学生遇到问题时的标准步骤：截图→描述"我看到了什么"→描述"我期望看到什么"→发给AI→按建议操作
    - 教师角色：引导学生走流程，不需要理解技术细节
    - 升级机制：AI解决不了 → 呼叫远程TA → TA远程协助
    - 提供具体的求助话术模板（"我在做X，但是出现了Y，我期望的是Z"）
    - 教师如何判断"这个问题需要叫TA"的决策树
  - **远程 TA 协作机制设计**：
    - TA 的角色定义（不是替学生写代码，而是引导学生跟AI更好地沟通）
    - 通信方式（微信群？飞书？屏幕共享工具？）
    - 响应时间预期
    - TA 工作量估算（每周大约几小时）
  - 混合年龄课堂管理策略（≥5条）
  - 教师如何评估学生作品（完全不需要理解代码的评估方法）

  **Must NOT do**:
  - 不能出现任何代码示例
  - 不能要求教师学习编程
  - 不能使用技术术语而不解释

  **Recommended Agent Profile**:
  - **Category**: `writing`
    - Reason: 教育培训类文档，核心是清晰的表达和实操指南
  - **Skills**: []

  **Parallelization**:
  - **Can Run In Parallel**: YES
  - **Parallel Group**: Wave 1 (with Tasks 1, 6, 8)
  - **Blocks**: Task 10
  - **Blocked By**: None

  **References**:
  - **Draft**: `.sisyphus/drafts/ai-edu-center.md` — 师资背景和担忧记录
  - **核心洞察**: 围棋老师具备的拆解能力、引导能力、设置挑战梯度的直觉，比编程经验更重要

  **Acceptance Criteria**:
  - [ ] 教师角色定义<300字
  - [ ] 2周自我培训计划，有每日具体活动
  - [ ] 课前准备清单模板
  - [ ] ≥15条FAQ，覆盖常见学生问题
  - [ ] 评估指南中零代码语法引用
  - [ ] ≥5条混合年龄课堂管理策略
  - [ ] 标准化 Debug 求助流程（含步骤图/流程图）
  - [ ] 求助话术模板≥3个
  - [ ] "需要叫TA"的决策树
  - [ ] 远程 TA 协作机制设计（角色/通信/响应时间/工作量）

  **QA Scenarios**:

  ```
  Scenario: 零代码术语检查
    Tool: Bash (grep)
    Steps:
      1. 在教师指南全文中搜索编程术语：function, variable, API, debug, compile, syntax, runtime, class, method, import
      2. 如果出现，检查是否有面向非技术人员的解释
    Expected Result: 零未解释的技术术语
    Evidence: docs/07-teacher-guide.md

  Scenario: 培训计划可执行性
    Tool: Bash (Agent 自检)
    Steps:
      1. 检查2周培训计划是否每天都有具体活动（不是"探索AI工具"而是"用Claude完成X任务"）
      2. 检查是否有自我评估清单
    Expected Result: 14天每天有具体任务；含自我评估清单
    Evidence: docs/07-teacher-guide.md

  Scenario: Debug 求助流程可用性
    Tool: Bash (Agent 自检)
    Steps:
      1. 模拟一个场景：学生做的网页打不开。检查流程是否指导教师和学生一步步解决
      2. 检查流程是否有可视化图示（流程图或步骤图）
      3. 检查"叫TA"的决策标准是否清晰（什么情况下升级）
      4. 检查求助话术模板是否具体可用
    Expected Result: 流程完整闭环；有可视化；升级标准明确；话术具体
    Evidence: docs/07-teacher-guide.md
  ```

  **Commit**: YES
  - Message: `docs: 教师指南与培训方案`
  - Files: `docs/07-teacher-guide.md`

- [ ] 8. 评估体系与作品集模板

  **What to do**:
  - 设计3级评估量表（未达到/进步中/已掌握），每阶段一个表：
    - 每个量表≤4个维度
    - 维度基于可观测行为，不基于技术知识
  - 设计学生作品集结构：
    - 收什么（每堂课的产出物、自我反思、同伴评价）
    - 怎么组织（按阶段分区）
    - 示例条目
  - 设计阶段转段评估协议：
    - 基于作品集展示，非考试
    - 学生演示能力，教师判断是否达标
    - 具体步骤清单
  - 设计家长沟通模板：
    - 每周进度快照（1页，视觉化，展示进步轨迹）
    - 每月详细报告
  - 学生自我反思提示（每阶段≥3个）
  - 同伴互评表（用于大带小配对）

  **Must NOT do**:
  - 评估维度不超过4个/阶段（保持简单）
  - 不基于代码质量评估（基于产品完成度和过程表现）
  - 不使用百分制或排名

  **Recommended Agent Profile**:
  - **Category**: `writing`
    - Reason: 教育评估框架设计，需要清晰的结构和易用的模板
  - **Skills**: []

  **Parallelization**:
  - **Can Run In Parallel**: YES
  - **Parallel Group**: Wave 1 (with Tasks 1, 6, 7)
  - **Blocks**: Task 10
  - **Blocked By**: None

  **References**:
  - **Draft**: `.sisyphus/drafts/ai-edu-center.md` — 讨论中确认的"大带小"协作模式
  - **围棋段位制**: 可参考围棋升段的评估理念——基于实战表现而非理论考试

  **Acceptance Criteria**:
  - [ ] 4个阶段各有一个量表（共4个量表）
  - [ ] 每个量表≤4维度 × 3级别
  - [ ] 作品集模板含分区说明和示例条目
  - [ ] 转段协议为步骤化清单
  - [ ] 家长报告模板为1页视觉化设计
  - [ ] 每阶段≥3个自我反思提示（共≥12个）
  - [ ] 含同伴互评表

  **QA Scenarios**:

  ```
  Scenario: 评估简洁性检查
    Tool: Bash (Agent 自检)
    Steps:
      1. 统计每个量表的维度数量，确认≤4
      2. 检查维度描述是否基于可观测行为（"学生能做X"而非"学生理解X"）
      3. 确认无百分制或排名机制
    Expected Result: 所有量表≤4维度；全部基于行为描述；无分数排名
    Evidence: docs/08-assessment-framework.md

  Scenario: 家长报告可读性
    Tool: Bash (Agent 自检)
    Steps:
      1. 检查家长报告模板是否≤1页
      2. 是否包含视觉化元素说明（进度条/图表/emoji等）
      3. 非教育背景的家长是否能3分钟内理解孩子的进步
    Expected Result: 模板≤1页；有视觉化设计；信息清晰直观
    Evidence: docs/08-assessment-framework.md
  ```

  **Commit**: YES
  - Message: `docs: 评估体系与作品集模板`
  - Files: `docs/08-assessment-framework.md`

- [ ] 2. 阶段一课程：对话者（Dialoguer）

  **What to do**:
  - 设计阶段一完整课程：教学生如何与AI进行有效对话，产出具体结果
  - 编写≥4个完整课时计划，每个包含：
    - 教学目标
    - 时间安排（分钟级）
    - 所需材料
    - 分步活动流程
    - 预期学生产出
    - 3级评估标准
    - 离线备用活动（网络中断时的替代方案）
  - 每个课时计划有小学版和中学版两个变体
  - 设计数字素养前置评估：学生会打字吗？会用浏览器吗？——不会的先补
  - 提供≥2组AI对话样例（好的提示 vs 差的提示对比）
  - 设计阶段结业项目：学生仅通过AI对话产出一个完整作品（故事集/攻略指南/调研报告等）
  - 工具选择：此阶段用 ChatGPT/Claude 的对话界面（网页版），不用编程工具
  - 体现与辩论课的能力迁移：清晰表达→清晰提示

  **Must NOT do**:
  - 此阶段不碰代码、不碰终端
  - 不教"提示词工程理论"（在做中学）
  - 课时计划不能假设教师懂技术

  **Recommended Agent Profile**:
  - **Category**: `writing`
    - Reason: 课程设计文档，需要结构化、详细的教学计划
  - **Skills**: []

  **Parallelization**:
  - **Can Run In Parallel**: YES
  - **Parallel Group**: Wave 2 (with Tasks 3, 4, 5)
  - **Blocks**: Tasks 9, 10
  - **Blocked By**: Task 1（需要L级定义和阶段入口/出口标准）

  **References**:
  - **依赖文档**: `docs/01-philosophy-and-levels.md` — L1-L2定义、阶段一出口标准
  - **Draft**: `.sisyphus/drafts/ai-edu-center.md` — 完整讨论记录

  **Acceptance Criteria**:
  - [ ] ≥4个完整课时计划
  - [ ] 每课含全部7个必需组件（目标/时间/材料/活动/产出/评估/离线备用）
  - [ ] 每课有小学版和中学版
  - [ ] ≥2组AI对话样例（好 vs 差）
  - [ ] 包含数字素养前置评估
  - [ ] 结业项目含评估量表
  - [ ] 标注本阶段估计总课时数

  **QA Scenarios**:

  ```
  Scenario: 课时计划完整性
    Tool: Bash (Agent 自检)
    Steps:
      1. 逐个检查课时计划，确认7个必需组件（目标/时间/材料/活动/产出/评估/离线备用）全部存在
      2. 检查每个课时是否有两个年龄变体
      3. 检查离线备用方案是否具体可执行
    Expected Result: 所有课时计划7组件齐全；全有年龄变体；离线方案具体
    Evidence: docs/02-phase1-dialoguer.md

  Scenario: 无代码泄漏检查
    Tool: Bash (grep)
    Steps:
      1. 搜索整个文档，确认无代码片段、终端命令或编程工具引用
      2. 确认工具引用仅限于网页版对话界面
    Expected Result: 零代码内容；仅引用网页版AI对话工具
    Evidence: docs/02-phase1-dialoguer.md
  ```

  **Commit**: YES
  - Message: `docs: 阶段一课程-对话者`
  - Files: `docs/02-phase1-dialoguer.md`

- [ ] 3. 阶段二课程：创造者（Creator）

  **What to do**:
  - 设计阶段二完整课程：教学生用AI创造可以运行的东西（网页、小游戏等）
  - 编写≥4个完整课时计划（同上7组件格式）
  - 准备≥3个预制启动项目（学生在此基础上修改），例如：
    - 一个简单的个人主页
    - 一个猜数字小游戏
    - 一个班级投票页面
  - 设计渐进式挑战序列：改颜色→加元素→改行为→组合功能
  - 引入"看代码但不写代码"的概念：学生看到AI产出的代码在运行，但不需要理解每一行
  - **渐进式代码素养启蒙**（核心新增）：
    - 教学生认识错误信息的"颜色"：红色=出错了，黄色=警告，绿色=成功
    - 教学生"代码像食谱"的类比：不需要发明食谱，但要能看出"盐放多了"
    - 简单的结构认知：一个文件=一个功能区域，文件名暗示了它的用途
    - 练习：给学生看AI修改前后的代码差异，让他们猜"AI改了什么"
  - 每课有年龄分层：小学生做简单版，中学生做进阶版
  - 结业项目：拿一个模板，定制成个人有意义的东西
  - 明确与阶段一的技能衔接："阶段一学的对话能力，现在用来造东西"
  - 工具：引入 Cursor/类似工具 或 Claude 的 artifact 功能

  **Must NOT do**:
  - 不教代码语法（但可以教"读"错误信息和识别结构）
  - 不要求学生手写代码
  - 修改挑战不能跳跃太大（80%学生应能完成）

  **Recommended Agent Profile**:
  - **Category**: `writing`
    - Reason: 课程设计文档
  - **Skills**: []

  **Parallelization**:
  - **Can Run In Parallel**: YES
  - **Parallel Group**: Wave 2 (with Tasks 2, 4, 5)
  - **Blocks**: Tasks 9, 10
  - **Blocked By**: Task 1

  **References**:
  - **依赖文档**: `docs/01-philosophy-and-levels.md` — L2-L3定义
  - **阶段一出口**: `docs/02-phase1-dialoguer.md` — 学生进入阶段二时已有的能力

  **Acceptance Criteria**:
  - [ ] ≥4个完整课时计划（含全部7组件）
  - [ ] ≥3个预制启动项目描述及修改挑战
  - [ ] 难度渐进清晰展示（easy→medium→hard）
  - [ ] 每课有年龄变体
  - [ ] 结业项目含评估量表
  - [ ] 含"从阶段一衔接"说明

  **QA Scenarios**:

  ```
  Scenario: 难度渐进合理性
    Tool: Bash (Agent 自检)
    Steps:
      1. 列出所有修改挑战，按难度排序
      2. 检查相邻挑战之间的难度跳跃是否合理
      3. 最简单的挑战是否足够简单（如：改一个颜色）
      4. 最难的挑战是否不超过阶段二能力范围
    Expected Result: 难度平滑渐进；最简单=改颜色/文字；最难不超过"组合多个修改"
    Evidence: docs/03-phase2-creator.md

  Scenario: 预制项目可用性
    Tool: Bash (Agent 自检)
    Steps:
      1. 每个预制项目是否有清晰的描述（是什么、长什么样、用什么技术）
      2. 修改挑战是否具体（不是"修改网页"而是"把背景色从蓝色改成你喜欢的颜色"）
    Expected Result: 项目描述清晰；挑战步骤具体到操作级别
    Evidence: docs/03-phase2-creator.md
  ```

  **Commit**: YES
  - Message: `docs: 阶段二课程-创造者`
  - Files: `docs/03-phase2-creator.md`

- [ ] 4. 阶段三课程：建造者（Builder）

  **What to do**:
  - 设计阶段三完整课程：教学生从教师提供的需求说明出发，用AI构建完整的简单应用
  - 编写≥4个完整课时计划（同上7组件格式）
  - 引入项目规划概念（我要造什么？给谁用？它做什么？）
  - 设计结构化AI工作流：规划→构建→测试→修复
  - 提供≥3个教师给定的项目需求文档（不同复杂度）
  - 教"用AI调试"：出了问题怎么跟AI描述问题（"它不能工作了，帮我看看" vs 精确描述现象）
  - **深化代码素养**（核心新增）：
    - 教学生读错误信息的"关键词"：哪个文件、哪一行、什么类型的错误
    - 教学生用截图+标注的方式向AI描述问题
    - 引入"标准化 Debug 流程"实践：截图→描述现象→描述期望→发给AI→执行建议→验证
    - 教学生判断"AI的修复是否解决了问题"（测试思维萌芽）
  - 年龄分层的项目复杂度
  - 结业项目：根据教师给定的需求文档，独立用AI构建一个完整的可用应用
  - 体现围棋战略思维→项目规划的能力迁移
  - 工具：引入 ClaudeCode 或 OpenCode

  **Must NOT do**:
  - 不要求学生手写代码
  - 不使用超出需求的技术复杂度
  - 项目需求文档不能用技术术语

  **Recommended Agent Profile**:
  - **Category**: `writing`
    - Reason: 课程设计文档
  - **Skills**: []

  **Parallelization**:
  - **Can Run In Parallel**: YES
  - **Parallel Group**: Wave 2 (with Tasks 2, 3, 5)
  - **Blocks**: Tasks 9, 10
  - **Blocked By**: Task 1

  **References**:
  - **依赖文档**: `docs/01-philosophy-and-levels.md` — L3定义
  - **围棋→项目规划**: 围棋中的布局→中盘→收官 类似于 项目规划→实现→完善

  **Acceptance Criteria**:
  - [ ] ≥4个完整课时计划（含全部7组件）
  - [ ] ≥3个项目需求文档模板（不同复杂度）
  - [ ] 结构化AI工作流指南（可视化/流程图格式）
  - [ ] AI调试对话指南（"怎么跟AI描述出了什么问题"）
  - [ ] 年龄变体
  - [ ] 结业项目含评估量表
  - [ ] 围棋→规划能力迁移说明

  **QA Scenarios**:

  ```
  Scenario: 项目需求文档可读性
    Tool: Bash (Agent 自检)
    Steps:
      1. 阅读每个项目需求文档，确认使用日常语言（非技术术语）
      2. 检查是否清晰描述了"做什么""给谁用""长什么样"
      3. 10岁的孩子是否能理解需求（至少简单版）
    Expected Result: 需求文档零技术术语；清晰的三要素描述；小学版可被10岁孩子理解
    Evidence: docs/04-phase3-builder.md

  Scenario: 调试指南实用性
    Tool: Bash (Agent 自检)
    Steps:
      1. 检查调试指南是否提供了具体的对话模板
      2. 是否有"差的描述 vs 好的描述"对比示例
    Expected Result: 含具体对话模板；含对比示例≥2组
    Evidence: docs/04-phase3-builder.md
  ```

  **Commit**: YES
  - Message: `docs: 阶段三课程-建造者`
  - Files: `docs/04-phase3-builder.md`

- [ ] 5. 阶段四课程：设计者（Designer）— L4 顶石阶段

  **What to do**:
  - 设计阶段四完整课程：L4顶石阶段，学生独立发现问题、设计方案、构建应用
  - 编写≥4个完整课时计划（同上7组件格式）
  - 设计"问题发现工作坊"：引导学生发现身边值得解决的真实问题
  - 引入用户中心设计基础：这是给谁用的？他们需要什么？
  - 独立项目规划与执行：学生自主定义项目、拆解步骤、用AI实现
  - 同伴评审与迭代：学生互相评审作品，提出改进建议
  - 设计年终 Demo Day 方案：
    - 向家长和朋友展示作品
    - 展示格式（3分钟演示 + 2分钟问答？）
    - 评审标准（创意性/实用性/完成度/表达能力）
  - 顶石项目：独立构思、规划、构建、展示一个完整应用
  - 体现辩论→需求收集、哲思→伦理设计的能力迁移
  - 设计"天才通道"：提前到达L4的学生的进阶活动
  - 年龄分层的项目复杂度期望

  **Must NOT do**:
  - 不限制项目主题（学生自选）
  - 不要求技术完美（重视过程和思考）
  - Demo Day 不搞排名（每个作品都值得展示）

  **Recommended Agent Profile**:
  - **Category**: `writing`
    - Reason: 课程设计文档，含展示活动策划
  - **Skills**: []

  **Parallelization**:
  - **Can Run In Parallel**: YES
  - **Parallel Group**: Wave 2 (with Tasks 2, 3, 4)
  - **Blocks**: Tasks 9, 10
  - **Blocked By**: Task 1

  **References**:
  - **依赖文档**: `docs/01-philosophy-and-levels.md` — L4定义、年龄分层标准
  - **辩论→需求收集**: 辩论中的论点构建 类似于 需求分析中的问题定义
  - **哲思→伦理**: 哲学思辨 应用于 "我的应用应不应该收集用户数据？"

  **Acceptance Criteria**:
  - [ ] ≥4个完整课时计划（含全部7组件）
  - [ ] 问题发现框架（适龄化）
  - [ ] 独立项目规划模板
  - [ ] 同伴评审协议
  - [ ] 顶石项目指南含评估量表 + 展示格式
  - [ ] 年龄变体
  - [ ] "天才通道"进阶活动
  - [ ] 辩论→需求、哲思→伦理 能力迁移说明
  - [ ] Demo Day 策划方案

  **QA Scenarios**:

  ```
  Scenario: 项目自主性验证
    Tool: Bash (Agent 自检)
    Steps:
      1. 检查顶石项目是否允许学生自选主题
      2. 评估标准是否侧重过程（规划/迭代/反思）而非技术完美
      3. Demo Day 是否有排名？（不应有）
    Expected Result: 学生自选主题；评估侧重过程；无排名
    Evidence: docs/05-phase4-designer.md

  Scenario: 天才通道完整性
    Tool: Bash (Agent 自检)
    Steps:
      1. 检查是否有明确的"提前达标学生"的识别标准
      2. 进阶活动是否≥3个
      3. 进阶活动是否真正有挑战性（不只是"多做一个项目"）
    Expected Result: 有识别标准；≥3个进阶活动；活动有实质性深度
    Evidence: docs/05-phase4-designer.md
  ```

  **Commit**: YES
  - Message: `docs: 阶段四课程-设计者`
  - Files: `docs/05-phase4-designer.md`

- [ ] 9. 挑战库（20+挑战，跨阶段，含年龄分层）

  **What to do**:
  - 创建≥20个独立挑战，可在各阶段灵活使用
  - 每个挑战包含：标题、目标阶段、难度（1-5）、预计时间、目标、材料、2个年龄变体（小学/中学）、成功标准、快速完成者的扩展活动
  - 挑战覆盖5个类别（每类≥3个）：
    - **创意类**: 故事、艺术描述、诗歌
    - **实用类**: 工具、计算器、日程表
    - **社交类**: 给家人看的东西、班级共享的东西
    - **游戏类**: 猜谜、问答、小游戏
    - **真实问题类**: 解决中心里的实际问题（食堂菜单查询？活动日历？）
  - 标记5个"展示挑战"：专门为家长 Demo Day 设计，产出有"哇"效果
  - 难度分布大致呈钟形：少量难度1和5，多数在2-4
  - 每个挑战可独立使用（不依赖前后挑战）

  **Must NOT do**:
  - 挑战不能依赖特定的技术知识
  - 不能只有代码类挑战（要多样化）
  - 展示挑战不能太复杂导致有失败风险

  **Recommended Agent Profile**:
  - **Category**: `writing`
    - Reason: 大量结构化的教育内容创作
  - **Skills**: []

  **Parallelization**:
  - **Can Run In Parallel**: NO
  - **Parallel Group**: Wave 3 (单独)
  - **Blocks**: Task 10
  - **Blocked By**: Tasks 2, 3, 4, 5（挑战需要对齐各阶段的课程内容）

  **References**:
  - **依赖文档**: `docs/02-phase1-dialoguer.md` 到 `docs/05-phase4-designer.md` — 各阶段的能力要求和活动类型
  - **Draft**: `.sisyphus/drafts/ai-edu-center.md` — 赵老师关于挑战设计的想法

  **Acceptance Criteria**:
  - [ ] ≥20个挑战
  - [ ] 每个挑战包含全部必需字段（标题/阶段/难度/时间/目标/材料/变体/标准/扩展）
  - [ ] 每阶段≥4个挑战
  - [ ] 每类别≥3个挑战（创意/实用/社交/游戏/真实问题）
  - [ ] 5个明确标记的展示挑战
  - [ ] 难度分布大致呈钟形

  **QA Scenarios**:

  ```
  Scenario: 挑战覆盖度检查
    Tool: Bash (Agent 自检)
    Steps:
      1. 统计每个阶段的挑战数量（≥4/阶段）
      2. 统计每个类别的挑战数量（≥3/类别）
      3. 统计难度分布（1-5各有多少）
      4. 确认5个展示挑战已标记
    Expected Result: 覆盖度达标；难度呈钟形分布；展示挑战标记清晰
    Evidence: docs/09-challenge-library.md

  Scenario: 挑战独立性验证
    Tool: Bash (Agent 自检)
    Steps:
      1. 随机抽取3个挑战，检查是否可以独立使用（不依赖其他挑战的产出）
      2. 检查是否有挑战隐含地假设了前置知识
    Expected Result: 所有挑战可独立使用；无隐含前置假设
    Evidence: docs/09-challenge-library.md
  ```

  **Commit**: YES
  - Message: `docs: 挑战库`
  - Files: `docs/09-challenge-library.md`

- [ ] 10. 集成审查与快速上手指南

  **What to do**:
  - 审查全部9份文档的内部一致性：
    - 阶段过渡逻辑是否通畅
    - L级定义在所有课程文档中是否一致使用
    - 成本模型与课程中的工具选择是否对齐
    - 教师指南是否覆盖了所有课程活动
    - 评估体系是否映射到所有课程产出
    - 挑战库引用的阶段是否正确
    - 文档间无矛盾
  - 产出：
    - 总目录（链接所有10份文档）
    - 1页快速上手指南（"先读这个"）
    - 需要与中心验证的假设/风险清单（≥10项）
  - 如发现矛盾，直接修复

  **Must NOT do**:
  - 不跳过任何文档的交叉检查
  - 不仅做表面检查（需要内容级验证）

  **Recommended Agent Profile**:
  - **Category**: `deep`
    - Reason: 需要深入阅读全部文档并进行细致的交叉比对
  - **Skills**: [`verification-before-completion`]
    - `verification-before-completion`: 确保在声称完成前进行严格验证

  **Parallelization**:
  - **Can Run In Parallel**: NO
  - **Parallel Group**: Wave FINAL (单独)
  - **Blocks**: None (最终任务)
  - **Blocked By**: Tasks 1-9（需要全部文档就绪）

  **References**:
  - **全部文档**: `docs/01-philosophy-and-levels.md` 到 `docs/09-challenge-library.md`

  **Acceptance Criteria**:
  - [ ] 总目录含所有10份文档的链接
  - [ ] 一致性检查清单全部通过（或发现的矛盾已修复）
  - [ ] 1页快速上手指南
  - [ ] 假设/风险清单≥10项
  - [ ] 无跨文档矛盾

  **QA Scenarios**:

  ```
  Scenario: 跨文档一致性
    Tool: Bash (Agent 深度阅读)
    Steps:
      1. 从 docs/01 提取L1-L4定义的关键词
      2. 在 docs/02-05 中搜索这些关键词，确认使用一致
      3. 从 docs/06 提取推荐的工具列表
      4. 在课程文档中确认引用的工具与成本模型一致
      5. 从 docs/08 提取评估维度
      6. 确认每个课程文档的产出物能被评估体系覆盖
    Expected Result: 术语一致；工具引用对齐；评估覆盖完整
    Evidence: docs/00-quick-start.md

  Scenario: 快速上手指南可用性
    Tool: Bash (Agent 自检)
    Steps:
      1. 快速上手指南是否≤1页
      2. 是否清晰指出"从哪里开始""按什么顺序读""立即要做什么"
      3. 一个今天刚接到任务的教师，能否在5分钟内理解整个体系的结构
    Expected Result: ≤1页；有明确的阅读路径和立即行动项；5分钟可理解
    Evidence: docs/00-quick-start.md
  ```

  **Commit**: YES
  - Message: `docs: 集成审查与快速上手指南`
  - Files: `docs/00-quick-start.md`

---

## Final Verification Wave

> Task 10（集成审查）作为最终验证。审查完成后，呈现结果给用户，获取明确的"确认"后才算完成。

---

## Commit Strategy

每个 Task 输出一个独立的 Markdown 文档到项目目录：

| Task | 输出文件 | 提交信息 |
|------|---------|---------|
| 1 | `docs/01-philosophy-and-levels.md` | docs: 课程哲学与能力等级定义 |
| 2 | `docs/02-phase1-dialoguer.md` | docs: 阶段一课程-对话者 |
| 3 | `docs/03-phase2-creator.md` | docs: 阶段二课程-创造者 |
| 4 | `docs/04-phase3-builder.md` | docs: 阶段三课程-建造者 |
| 5 | `docs/05-phase4-designer.md` | docs: 阶段四课程-设计者 |
| 6 | `docs/06-infrastructure-and-cost.md` | docs: 基础设施与成本模型 |
| 7 | `docs/07-teacher-guide.md` | docs: 教师指南与培训方案 |
| 8 | `docs/08-assessment-framework.md` | docs: 评估体系与作品集模板 |
| 9 | `docs/09-challenge-library.md` | docs: 挑战库 |
| 10 | `docs/00-quick-start.md` | docs: 集成审查与快速上手指南 |

---

## Success Criteria

### 最终检查清单
- [ ] 全部10份文档已产出
- [ ] 所有课时计划通过"非技术教师可执行"检查
- [ ] 每个活动/挑战有小学版和中学版
- [ ] 成本模型可验证，基于真实定价
- [ ] 阶段过渡基于作品集检查点，非时间推进
- [ ] "Must NOT Have" 清单中的所有项均未出现
- [ ] 全套文档无内部矛盾（集成审查确认）
- [ ] 用户确认方案可行
