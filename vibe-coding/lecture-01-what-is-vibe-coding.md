# Lecture 1: 什么是 Vibe Coding？
# Lecture 1: What Is Vibe Coding?

> **教学目标 / Learning Objectives**
> - 用自己的话解释 vibe coding 是什么 / Explain vibe coding in your own words
> - 理解它和传统编程的本质区别 / Understand how it differs from traditional coding
> - 知道它能做什么、不能做什么 / Know what it can and cannot do

---

## Feynman Step 1：先用最简单的话解释
## Feynman Step 1: Explain It Simply

> 🎯 目标：假设你在向一个从没写过代码的同学解释，怎么说？

**简单解释 / Simple Explanation：**

Vibe Coding 是一种新的造软件的方式：**你说你想要什么，AI 帮你写代码。**

你不需要懂代码。你只需要描述你的想法——用中文、用英文都行——AI 会把代码写出来，然后你看看效果，再继续描述下一步要改什么。就这样来回几轮，一个 App 就出来了。

这个词是 **Andrej Karpathy**（前特斯拉 AI 负责人，OpenAI 联合创始人）在 2025 年 2 月发明的。他原话是：

> *"There's a new kind of coding I call 'vibe coding', where you fully give in to the vibes, embrace exponentials, and forget that the code even exists. I just see stuff, say stuff, run stuff, and copy paste stuff, and it mostly works."*
>
> *（有一种新的编程方式，我叫它"vibe coding"：你完全顺着感觉走，忘掉代码的存在。我只是看看效果、说说想法、跑跑程序、复制粘贴，然后——它基本就能用了。）*

**类比 / Analogy：**

想象你要建一栋房子：

- **传统编程** = 你自己当工人：买砖头、和水泥、砌墙、装电路……每一步都要自己动手，你必须懂建筑。
- **Vibe Coding** = 你当设计师：你告诉 AI "我想要一个两层的房子，客厅朝南，要有一个大窗户"，AI 是建筑队，自动把房子盖好。你只管验收、提意见。

---

## Feynman Step 2：找漏洞 — 哪里说不清楚？
## Feynman Step 2: Identify the Gaps

> 🎯 目标：诚实面对"我刚才哪里说得模糊了？"

| 漏洞 | 刚才怎么说的 | 真正没说清楚的问题 |
|------|------------|-----------------|
| 1 | "AI 帮你写代码" | 是哪个 AI？它怎么知道怎么写？ |
| 2 | "基本就能用" | 什么时候不能用？会出什么问题？ |
| 3 | "你不需要懂代码" | 真的吗？完全不懂代码也行？ |
| 4 | "来回描述几轮" | 怎么描述才有效？随便说话就行吗？ |

**遇到的专业词 / Jargon check：**

| 词汇 | 我能简单解释吗？ |
|------|---------------|
| LLM (大语言模型) | 能 / Yes |
| Agent（智能体） | 模糊 / Fuzzy |
| Prompt（提示词） | 能 / Yes |
| MCP | 不能 / No |

---

## Feynman Step 3：填漏洞 — 深入讲透
## Feynman Step 3: Fill the Gaps

### 漏洞 1：是哪个 AI？它怎么知道怎么写代码？

**问题：** AI 写代码的原理是什么？

**答案：**

背后的 AI 叫做 **LLM（大语言模型）**，比如 Claude（Anthropic 出的）、GPT-4（OpenAI）、Gemini（Google）。

它们是怎么学会写代码的？用了一个超级简单的原理：**海量阅读 + 预测下一个词**。

它们读了 GitHub 上数十亿行代码、无数文档、Stack Overflow 的问答……读完之后，给它一个开头，它就能预测"接下来最可能出现什么代码"——就像你背了一万首古诗之后，别人说"床前明月光"你就能接"疑是地上霜"。

但注意：它不是"理解"代码，它是在做**极度复杂的模式匹配**。这也是为什么它有时候会犯很蠢的错误。

**Vibe Coding 工具**（比如 Google AI Studio、Cursor、Claude Code）在 LLM 外面加了一层壳，让 AI 不只是给你回答，而是可以：
- 直接创建文件
- 运行程序
- 看到报错然后自己修
- 反复迭代直到能跑起来

---

### 漏洞 2："基本就能用" — 什么时候不能用？

**问题：** Vibe Coding 的极限在哪里？

**答案：**

Vibe Coding **擅长的事：**
- ✅ 做出一个能看、能用的原型（Prototype）
- ✅ 简单的个人网站、工具、游戏
- ✅ 把一个想法快速变成可以演示的东西
- ✅ 非技术人员验证产品想法

Vibe Coding **容易出问题的地方：**
- ❌ 安全性（登录系统、支付、用户数据——AI 写的代码可能有严重漏洞）
- ❌ 复杂业务逻辑（规则很多、边界情况很多时，AI 会漏掉很多）
- ❌ 大型项目（代码越来越多，AI 就越来越容易"忘记"之前的结构）
- ❌ 性能要求高的场景（AI 写的代码通常不考虑优化）

**真实案例（2026年）：** Amazon 曾因为用 AI 辅助修改代码、但没有认真审查，导致了**生产环境重大故障**。这是 vibe coding 失控的典型案例。

---

### 漏洞 3："真的不需要懂代码吗？"

**问题：** 完全不懂代码的人能用 Vibe Coding 做出真实产品吗？

**答案：这取决于你的目标。**

Vibe Coding 是一个**光谱（Spectrum）**，不是非此即彼：

```
纯 Vibe Coding ←————————————→ AI 辅助的专业开发
    
[完全不看代码]              [偶尔看代码，大量用 AI]
[适合：个人项目/原型]       [适合：生产级产品]
[风险：安全漏洞/技术债]     [风险：几乎没有]
```

- **Karpathy 的极端版本**：完全不看代码，顺着感觉走。适合实验性项目。
- **现实中更常见的版本**：有一点点编程基础，用 AI 大幅提速，但还是会审查关键代码。

Karpathy 后来自己也说，这个概念应该进化为 **"Agentic Engineering"（智能体工程）**——即你像工程师一样思考，但让 AI Agent 来执行。

---

### 漏洞 4："怎么描述才有效？"

**问题：** 随便说话 AI 就能理解吗？有没有技巧？

**答案：**

这就是为什么 **Prompt Engineering（提示词工程）** 很重要。简单来说：

| 差的提示词 | 好的提示词 |
|-----------|----------|
| "做个网站" | "做一个个人作品集网站：深色背景、顶部导航栏有 Home/About/Projects 三个菜单、首页有一个全屏 Hero 区域显示我的名字和一句介绍语、下面有 3 个项目卡片" |
| "修一下这个 bug" | "第 42 行报错：TypeError: cannot read property 'name' of undefined。原因是 user 对象有时候是 null，请加一个判空处理" |

**好提示词的原则（Lecture 3 会深讲）：**
1. **具体** — 描述你想要的结果，而不是过程
2. **有背景** — 告诉 AI 这个项目是什么、用什么技术
3. **一次一件事** — 不要一次让 AI 做十件事
4. **会迭代** — 第一次不完美没关系，描述哪里不对，继续改

---

## Feynman Step 4：精炼解释 — 最终版本
## Feynman Step 4: Refined Explanation

**最终简单解释：**

Vibe Coding 是一种用**自然语言驱动 AI 写代码**的新方式。你描述你想要什么，AI 写代码、运行、报错、自己修，你只看结果、提意见。它的核心是：**把"怎么写"的工作交给 AI，你只关注"想要什么"。**

它存在于一个光谱上——从完全不碰代码的非技术人员，到用 AI 大幅提速的专业工程师。它很强大，但在安全性、复杂业务逻辑、大型项目上有明显的风险，需要谨慎。

**精炼类比：**

Vibe Coding 像是**当一个电影导演**，而不是摄影师：

- 摄影师（传统程序员）要自己操控摄像机、调光圈、设置焦距。
- 导演（Vibe Coder）说："我要一个从低角度仰拍的镜头，光线昏暗，让观众感到压抑"——然后 AI 摄影师去执行。

一个**好导演**不需要自己操摄像机，但他必须**懂得什么样的镜头能表达什么感觉**，才能给出好的指令。完全不懂电影语言的导演，拍出来的东西会一塌糊涂。

**核心要点：**
1. Vibe Coding = 用自然语言驱动 AI 写代码，你关注"要什么"，AI 负责"怎么做"
2. 它是一个光谱，从"完全不看代码"到"AI 辅助的专业开发"
3. 强大但有边界：安全性、复杂逻辑、大项目是危险地带
4. 提示词质量直接决定结果质量

> **30 秒版本：**
> "Vibe Coding 就是你跟 AI 说你想要什么，它帮你把代码写出来。你不用懂代码，但你要懂得怎么描述你想要的东西。它适合做原型和个人项目，但对安全性要求高的场景要小心。"

---

## 🔬 课后练习
## 🔬 Hands-on Practice

### 练习 1-A：体验纯 Vibe Coding（约 20 分钟）

**步骤：**
1. 打开 [Google AI Studio](https://aistudio.google.com)（免费，无需安装，用 Google 账号登录即可）
2. 输入以下提示词：
   > *"帮我做一个个人主页：深色背景，顶部显示我的名字（随便取一个），下面有一段自我介绍，再下面有 3 个作品卡片（内容随意），底部有邮箱联系方式。"*
3. 看 AI 生成的结果
4. **不要修改代码**，只通过自然语言给反馈，至少改 3 轮：
   - 第 1 轮："把主题色改成紫色系"
   - 第 2 轮："卡片加上鼠标悬停效果"
   - 第 3 轮：你自己提一个要求

**记录下来：**
- ✍️ 哪些地方 AI 做得超出你的预期？
- ✍️ 哪些地方 AI 做错了或没理解你的意思？
- ✍️ 你有没有不得不"解释很多遍"的地方？

---

### 练习 1-B：Feynman 自测（约 10 分钟）

用自己的话，向一个完全不懂技术的人解释：
> **"Vibe Coding 和用搜索引擎有什么不同？为什么它更强大，或者更危险？"**

试着写 3-5 句话，不用任何专业词汇。

---

### 练习 1-C：思考题（课堂讨论）

> 如果 Vibe Coding 真的让所有人都能"造软件"，会发生什么？
> - 对程序员这个职业有什么影响？
> - 会不会出现很多"看起来能用但其实有严重 bug"的软件？
> - 你愿意用一个"完全由 AI 写的"App 来管理你的银行账户吗？为什么？

---

## 📚 延伸阅读
## 📚 Further Reading

| 资源 | 类型 | 难度 | 简介 |
|------|------|------|------|
| [Andrej Karpathy 原推](https://x.com/karpathy/status/1886192184808149383) | 推文 | ⭐ | Vibe Coding 概念的诞生 |
| [Wikipedia: Vibe Coding](https://en.wikipedia.org/wiki/Vibe_coding) | 百科 | ⭐ | 概念定义和历史溯源 |
| [Fireship 解说视频](https://www.youtube.com/watch?v=Tw18-4U7mts) | 视频 | ⭐ | 6分钟搞懂 Vibe Coding |
| [NYT: AI and Vibecoding](https://www.nytimes.com/2025/02/27/technology/personaltech/vibecoding-ai-software-programming.html) | 文章 | ⭐⭐ | 普通人用 Vibe Coding 造软件的真实故事 |
| [Vibe coding is passé](https://thenewstack.io/vibe-coding-is-passe/) | 文章 | ⭐⭐ | Karpathy 对概念的进化思考 |

---

> **下一讲预告 →** Lecture 2：工具全景图 — 市面上有 50+ 款 Vibe Coding 工具，它们有什么区别？你应该用哪个？
