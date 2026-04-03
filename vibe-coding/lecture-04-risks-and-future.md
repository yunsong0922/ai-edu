# Lecture 4: 风险、边界与未来
# Lecture 4: Risks, Limits, and the Future

> **教学目标 / Learning Objectives**
> - 识别 Vibe Coding 的真实风险，特别是安全性和技术债 / Identify real risks, especially security and technical debt
> - 理解 Vibe Coding 对编程职业和开源生态的影响 / Understand the impact on programming careers and open source
> - 形成对这项技术的批判性视角，知道什么时候该用、什么时候该谨慎 / Develop critical judgment about when to use it and when to be cautious

---

## Feynman Step 1：先用最简单的话解释
## Feynman Step 1: Explain It Simply

> 🎯 目标：作为一个批判性思考者，我们不光学怎么用，还要懂为什么不能乱用。

**简单解释：**

Vibe Coding 很强大，但它有一个根本性的问题：

> **你在用你不完全理解的代码。**

就像你能开车但不懂发动机原理——大多数时候没问题，但出了故障你不知道为什么，也不知道怎么修。在软件里，这会导致：

- 🔓 **安全漏洞**：AI 写的登录系统可能被黑客轻松攻破
- 🗑️ **技术债**：代码能跑，但乱得无法维护
- 🌐 **开源危机**：大量低质量 AI 代码涌入 GitHub
- 😱 **真实事故**：Amazon 2026 年因 AI 辅助代码导致生产环境崩溃

**类比：**

Vibe Coding 的风险像 **外卖食物过敏**：

- 大多数时候，外卖很方便，你享用美食
- 但如果你对花生过敏，而厨师（AI）不知道这件事，而且你也没看配料表（没看代码）
- 有一天，餐里加了花生酱——你不知道，吃了，出了大问题
- 自己做饭（传统编程）：你知道每一个配料，不会出现这种事

---

## Feynman Step 2：找漏洞
## Feynman Step 2: Identify the Gaps

| 漏洞 | 刚才怎么说的 | 真正没说清楚的问题 |
|------|------------|-----------------|
| 1 | "安全漏洞" | 具体有什么漏洞？怎么发生的？能举个例子吗？ |
| 2 | "技术债" | 技术债是什么？为什么 AI 代码会产生更多技术债？ |
| 3 | "开源危机" | Vibe Coding 怎么影响开源社区？ |
| 4 | "Amazon 崩了" | 具体发生了什么？怎么避免？ |
| 5 | 没说 | 对于要学编程的学生，Vibe Coding 是机会还是威胁？ |

---

## Feynman Step 3：填漏洞
## Feynman Step 3: Fill the Gaps

### 漏洞 1：具体有什么安全漏洞？

**AI 最常制造的安全问题（附真实场景）：**

**① SQL 注入（SQL Injection）**

你让 AI 做一个"搜索商品"功能，AI 写出这样的代码：

```python
# AI 可能写出的危险代码（伪代码）
query = "SELECT * FROM products WHERE name = '" + user_input + "'"
```

如果用户输入的不是"苹果"，而是 `苹果'; DROP TABLE products; --`，数据库就会把整个商品表删掉。这就是 SQL 注入——你看不懂代码，不知道这里有问题。

**② 密码明文存储**

你让 AI 做用户注册，AI 直接把密码存到数据库：

```python
# 危险写法
db.save(username=user, password=input_password)  # 明文密码！

# 正确写法（要用哈希加密）
db.save(username=user, password=bcrypt.hash(input_password))
```

用户数据库一旦泄露，所有密码都裸奔。

**③ 没有权限验证**

AI 做了"删除订单"的 API，但忘记检查"当前用户是否有权限删这个订单"——任何人知道订单 ID 都能删别人的订单。

**为什么 AI 会犯这些错误？**

- AI 是在写"看起来正确"的代码，不是在思考安全边界
- 训练数据里有大量不安全的代码示例
- AI 无法真正理解"黑客会怎么攻击这个"

**实用原则：**

```
"凡是涉及以下场景，必须人工审查或找安全专家验收：
 - 用户登录/注册/密码
 - 支付/金融数据
 - 用户上传文件
 - 数据库的读写删除
 - 对外暴露的 API"
```

---

### 漏洞 2：技术债是什么？AI 代码为什么容易产生技术债？

**技术债（Technical Debt）简单解释：**

代码能跑，但写得很乱，以后每次修改都会越来越难、越来越痛苦。就像**借了钱**——你现在能花，但以后要还利息，利息越滚越多。

**AI 代码常见的技术债问题：**

| 问题 | 表现 | 后果 |
|------|------|------|
| 代码重复 | 同样的逻辑写了 5 次 | 改一个地方要改 5 个地方，容易漏 |
| 命名混乱 | `data1`、`temp`、`result2` | 3 个月后没人（包括你）看得懂 |
| 没有错误处理 | 只写了成功的情况，失败时直接崩溃 | 用户看到白屏或奇怪报错 |
| 结构混乱 | 所有代码堆在一个文件里 | 项目大了之后，加任何新功能都是噩梦 |
| 没有注释 | AI 生成的代码没有解释 | 无法维护，只能推倒重来 |

**为什么 AI 容易产生技术债？**

- AI 优化的是"当前请求是否能完成"，不是"这段代码6个月后容不容易维护"
- 每次对话独立，AI 不记得上次怎么设计架构
- AI 倾向于"能跑就行"，不会主动重构

**对付技术债的方法：**

```
1. 定期让 AI "代码审查"：
   "请检查 /src 下的代码，找出重复逻辑、命名不清晰的地方，给出改进建议"

2. 早期建立好的结构：
   "这个项目的代码要遵循 MVC 架构，给我建议一个文件结构"

3. 使用代码质量检查工具：
   pyscn（Python 专用）、ESLint（JavaScript）——AI 写完，自动检查
```

---

### 漏洞 3：Vibe Coding 怎么影响开源社区？

**背景：**

2026 年的 Hackaday 文章标题是：[**"How Vibe Coding Is Killing Open Source"**](https://hackaday.com/2026/02/02/how-vibe-coding-is-killing-open-source/)

**问题在哪里？**

① **GitHub 被 AI 垃圾淹没**

Vibe Coding 让每个人都能生成大量代码并提交到 GitHub。问题是：

- 大量低质量、重复的代码库出现
- 自动生成的 "贡献" 充斥开源项目的 PR（Pull Request）
- 人工维护者被大量垃圾 PR 淹没，精力耗尽

② **AI 训练数据的污染循环**

```
AI 生成代码 → 代码被提交到 GitHub → 
下一代 AI 用这些代码训练 → 
AI 学到了更多 AI 风格的代码（质量堪忧） → 
AI 生成更多代码...
```

这个循环如果不受控，AI 代码的质量会越来越差，因为它在"学习自己"。

③ **开源维护者的困境**

真正的开源贡献者（花时间写高质量代码、解决真实问题的人）正在被 AI 生成的"贡献"边缘化，社区维护成本急剧上升。

**这不是说 Vibe Coding 本身是错的**，而是说——**不加思考地用 AI 生成代码并发布到公共平台，会有负外部效应**。

---

### 漏洞 4：Amazon 到底发生了什么？

**事件（2026 年 10 月）：** Vibe-coded changes caused major outages at Amazon

**重建场景（根据公开报道推测）：**

```
工程师 A 用 AI 辅助写了一个"看起来没问题"的配置更改
→ 没有仔细 code review（代码审查）
→ 代码进了主分支，被部署到生产环境
→ 某个 AI 写的边界条件处理有误
→ 在高并发场景下触发
→ 服务连锁崩溃
→ 影响数百万用户
```

**核心教训：**

1. **AI 生成的代码需要和人写的代码一样严格审查**——甚至更严格，因为 AI 不会理解业务上下文
2. **"它生成了、能跑"不等于"它是对的"**
3. **越是核心系统，越要保持怀疑态度**

---

### 漏洞 5：对学生来说，Vibe Coding 是机会还是威胁？

**这是最重要的问题，诚实面对。**

**机会层面 ✅：**

- 降低了"起步门槛"——有想法的人可以快速验证，不再被技术挡在门外
- 加速了从"想法"到"产品"的速度
- 让开发者可以专注更高层的设计和决策，把重复性工作交给 AI
- Karpathy 的最新观点：未来的开发者是 **"Agentic Engineer"**——像指挥官一样调度 AI 代理，而不是自己写每一行

**威胁层面 ⚠️：**

- 如果**只会 Vibe Coding，不懂底层原理**，就像只会开自动档但不懂引擎——出了问题完全无法判断
- 学生时期依赖 AI 完成作业，会失去对基础概念的真正理解
- 未来职场：会 Vibe Coding 的人多了，真正能**理解、审查、修复** AI 代码的人反而更稀缺、更值钱

**Red Hat 工程师的"不舒服的真相"（2026 年文章）：**

> *"Vibe coding is great for getting 80% of the way there. The remaining 20% — the edge cases, the security, the performance, the maintainability — still requires a real engineer."*
>
> *（Vibe coding 能帮你完成 80% 的工作。剩下的 20%——边界情况、安全性、性能、可维护性——仍然需要真正的工程师。）*

**结论：**

```
最佳路径（对学生）：

学阶段 1：理解基础 → 为什么代码要这样写，数据流是怎么走的
学阶段 2：学会用 Vibe Coding 工具 → 快速构建，放大能力
学阶段 3：学会审查 AI 代码 → 识别漏洞，判断质量
学阶段 4：成为 Agentic Engineer → 调度多个 AI Agent，像架构师一样思考

危险路径（避免）：
跳过阶段 1 和 3，直接 Vibe Coding 一切，
对生成的代码毫无判断力。
```

---

## Feynman Step 4：精炼解释
## Feynman Step 4: Refined Explanation

**最终简单解释：**

Vibe Coding 的最大风险是：**你在使用你不完全理解的代码**。这会带来安全漏洞（SQL 注入、密码明文存储、权限漏洞）、技术债（乱得没法维护的代码库）、以及更宏观的影响（开源社区质量下降、生产事故）。

这些风险不代表 Vibe Coding 是坏的，而是说它需要**判断力**——知道什么时候可以放手让 AI 做，什么时候必须人工介入审查。对学生而言，最好的策略是：先打好基础，再用 Vibe Coding 放大能力，而不是把它当成逃避学习的捷径。

**精炼类比：**

Vibe Coding 的风险就像**用自动翻译写合同**：

- 自动翻译能翻出 80% 的意思，省了大量时间
- 但法律合同里，每个词的精确含义都至关重要
- 如果你不懂法律语言，你不会知道哪里翻错了
- 签了一份有问题的合同，后果可能是灾难性的
- 正确做法：用自动翻译打草稿，但必须让懂法律的人逐字审查

**核心要点：**
1. 安全性是 Vibe Coding 的最大盲区——凡是涉及登录、支付、用户数据，必须人工审查
2. 技术债是慢性病——AI 代码能跑，但维护成本可能很高
3. Amazon 案例告诉我们：审查 AI 代码和审查人类代码一样重要
4. 对学生：先打基础，再用 Vibe Coding；跳过基础只会 Vibe Coding 是危险的

> **30 秒版本：**
> "Vibe Coding 最大的陷阱是：代码跑了，但你不知道它是不是对的，更不知道它安不安全。AI 写的代码能被 SQL 注入、密码可能明文存储、权限可能根本没验证。原则是：个人项目随意，但凡有真实用户数据，就必须亲自审查关键代码。"

---

## 🔬 课后练习
## 🔬 Hands-on Practice

### 练习 4-A：安全审计实验（约 30 分钟）

1. 用任意 Vibe Coding 工具生成一个**包含登录功能**的简单 App
2. 让 AI 审查它自己生成的代码：
   > *"请审查这段登录代码，找出所有潜在的安全漏洞，包括 SQL 注入、密码存储方式、权限验证等问题，并给出修复建议"*
3. 记录：
   - AI 找到了哪些问题？
   - AI 有没有漏掉明显的问题？（对比搜索"常见 web 安全漏洞"）
   - 你自己能看懂这些漏洞吗？

**思考：** 如果这是一个真实的 App，有 1000 个真实用户，你敢不敢上线？

---

### 练习 4-B：技术债检测（约 20 分钟）

找到你在 Lecture 3 练习里做的"学生成绩记录器"项目，让 AI 做一次代码审查：

> *"请检查整个项目的代码质量：找出重复代码、命名不清晰、缺少错误处理、结构混乱的地方，列出具体问题和改进建议"*

数一数：AI 找到了多少个技术债问题？有没有你已经注意到的？有没有你完全没想到的？

---

### 练习 4-C：辩论——Vibe Coding 是福还是祸？（课堂讨论，约 30 分钟）

分成两组，分别论证：

**正方：** "Vibe Coding 是编程民主化的里程碑，它让更多人能参与创造，是好事"

**反方：** "Vibe Coding 制造了大量不安全、不可维护的软件，正在破坏整个软件行业"

各组准备 3 个论点，要有具体例子支撑。

讨论后，写下你个人的结论（不需要站队，可以是"在什么条件下是好事，什么条件下是坏事"）。

---

### 练习 4-D：未来职业思考（个人作业，约 15 分钟）

思考并写下你的答案：

1. 如果 Vibe Coding 让每个人都能造 App，**那程序员还有什么独特价值**？
2. 你认为 5 年后，哪些编程技能会更值钱（因为 AI 不擅长）？哪些会变得不那么重要？
3. 如果你是一个要给 10 年后就业市场做准备的中学生，你现在应该重点学什么？

---

## 📚 延伸阅读
## 📚 Further Reading

| 资源 | 类型 | 难度 | 简介 |
|------|------|------|------|
| [Vibe-Coding Triggered Amazon Outages](https://belitsoft.com/news/vibe-coding-amazon-outage-20261003) | 新闻 | ⭐ | Amazon 生产事故真实案例 |
| [How Vibe Coding Is Killing Open Source](https://hackaday.com/2026/02/02/how-vibe-coding-is-killing-open-source/) | 文章 | ⭐⭐ | 批判性分析 Vibe Coding 对开源的影响 |
| [The Uncomfortable Truth About Vibe Coding](https://developers.redhat.com/articles/2026/02/17/uncomfortable-truth-about-vibe-coding) | 文章 | ⭐⭐ | Red Hat 工程师的诚实反思 |
| [Vibe Coding Could Cause Catastrophic Explosions in 2026](https://thenewstack.io/vibe-coding-could-cause-catastrophic-explosions-in-2026/) | 文章 | ⭐⭐ | 技术风险预测分析 |
| [What is vibe coding? A computer scientist explains](https://theconversation.com/what-is-vibe-coding-a-computer-scientist-explains-what-it-means-to-have-ai-write-computer-code-and-what-risks-that-can-entail-257172) | 文章 | ⭐⭐ | 学术视角的风险分析 |
| [Vibe Engineering (Manning)](https://www.manning.com/books/vibe-engineering) | 书 | ⭐⭐⭐ | 系统性地讲如何负责任地做 AI 辅助开发 |

---

> **下一讲预告 →** Final Project：把所有学到的东西整合起来，从零开始完整地做一个真实项目——用完整工作流、好的提示词、AGENTS.md、频繁 commit、并在最后做一次安全审查。
