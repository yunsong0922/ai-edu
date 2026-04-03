# Lecture 2: 工具全景图 — 如何选对你的武器
# Lecture 2: The Tools Landscape — Choose Your Weapon

> **教学目标 / Learning Objectives**
> - 了解 Vibe Coding 工具的 4 大类别及其区别 / Understand the 4 categories of vibe coding tools
> - 能根据自己的背景和目标选对工具 / Match tools to your background and goals
> - 理解工具背后的关键概念：Agentic、MCP、本地/云端 / Understand key concepts: Agentic, MCP, local vs cloud

---

## Feynman Step 1：先用最简单的话解释
## Feynman Step 1: Explain It Simply

> 🎯 目标：市面上有 50+ 款工具，你怎么不被淹没？

**简单解释：**

Vibe Coding 工具可以按**你在哪里用、你有多少技术背景**分成 4 大类：

| 类型 | 在哪里用 | 面向谁 | 代表工具 |
|------|---------|-------|---------|
| 🌐 **浏览器工具** | 打开网页就能用 | 非技术人员、快速原型 | **Google AI Studio**、Bolt.new、v0 |
| 💻 **AI IDE** | 下载安装的代码编辑器 | 开发者，要速度+控制 | Cursor、Windsurf、Zed |
| ⌨️ **命令行工具** | 在终端里运行 | 高级工程师 | Claude Code、aider |
| 🧩 **插件/扩展** | 装在已有编辑器里 | 不想换编辑器的开发者 | GitHub Copilot、Cline |

**类比：**

选 Vibe Coding 工具就像选**交通工具**：

- 🌐 浏览器工具 = **滴滴/Uber**：你说去哪，它载你到，全程不用自己开车
- 💻 AI IDE = **有辅助驾驶的特斯拉**：你坐在驾驶位，车会帮你大部分操作，但你可以随时接管
- ⌨️ 命令行工具 = **摩托车+导航**：速度最快、灵活性最高，但你得会骑
- 🧩 插件 = **给你现有的车加装辅助功能**：不换车，直接升级

---

## Feynman Step 2：找漏洞
## Feynman Step 2: Identify the Gaps

| 漏洞 | 刚才怎么说的 | 真正没说清楚的问题 |
|------|------------|-----------------|
| 1 | "浏览器工具适合非技术人员" | 专业开发者用浏览器工具有价值吗？ |
| 2 | "命令行工具适合高级工程师" | CLI 比 IDE 究竟强在哪？ |
| 3 | 列了一堆工具名字 | 同类工具之间怎么选？比如 Cursor vs Windsurf？ |
| 4 | 没提 Task Management 和 Documentation 工具 | 这些是干什么的？ |

**遇到的专业词：**

| 词汇 | 能简单解释吗？ |
|------|-------------|
| Agentic（智能体模式） | 模糊——AI 能"主动行动"？ |
| MCP（模型上下文协议） | 不能——经常出现，但不知道是什么 |
| Git worktree | 不能——和并行 AI 代理有关？ |
| Local vs Cloud model | 能——本地运行 vs 云端 API |

---

## Feynman Step 3：填漏洞
## Feynman Step 3: Fill the Gaps

### 漏洞 1：专业开发者用浏览器工具有价值吗？

**答：绝对有，但用法不同。**

浏览器工具不等于"玩具"。它们的价值在于**速度**——快速验证一个想法，不需要配置环境、不需要部署。

典型工作流：
```
想法 → [v0/Bolt 浏览器快速原型，10分钟] → 看效果 → 
导出真实代码 → [Cursor/Windsurf 精炼，认真开发] → 上线
```

**Firebase Studio**（Google 出的）和 **Emergent** 已经面向专业开发者，支持构建生产级全栈应用。

结论：**浏览器工具 = 原型阶段**，不管你是不是技术人员都有用。

---

### 漏洞 2：命令行工具比 AI IDE 强在哪？

**答：CLI 更适合"深度操作大型项目"。**

| 维度 | AI IDE（Cursor 等） | CLI（Claude Code 等） |
|------|-------------------|--------------------|
| 上手难度 | 低，图形界面 | 高，要懂命令行 |
| 项目规模 | 中小型项目更顺手 | 大型项目更适合 |
| 自动化能力 | 一般 | 极强（可脚本化、可定时执行） |
| 与其他工具集成 | 有限 | 极强（管道、组合、CI/CD） |
| 适用人群 | 大多数开发者 | 住在终端里的工程师 |

**Claude Code** 是目前评价最高的 CLI 工具，能理解整个代码库、自动执行 Git、修改多个文件、运行测试——全程不需要你打开任何界面。

---

### 漏洞 3：同类工具之间怎么选？

**一个决策树：**

```
我是谁？
│
├── 完全不懂代码，只想把想法变成产品
│   └── → **Google AI Studio**（首选，完全免费）或 Bolt.new
│
├── 懂一点代码，想快速做原型
│   └── → v0（最适合 Web/React）或 Replit（最适合有后端的项目）
│
├── 会写代码，想用 AI 提速
│   ├── 用 VS Code → Cursor（综合最强）
│   ├── 用 JetBrains → Cursor / Junie
│   └── 用 Neovim → avante.nvim / CodeCompanion.nvim
│
├── 会写代码，喜欢在终端工作
│   └── → Claude Code（最推荐）或 aider（Git 集成最好）
│
└── 不想换工具，只想加 AI 能力
    ├── VS Code 用户 → GitHub Copilot 或 Cline
    └── 其他编辑器 → Continue（支持最广）
```

**几个值得单独说的工具：**

- 🔥 **Cursor**：目前最流行的 AI IDE，被大量职业开发者使用。Cloud Agents 模式可以完全自主完成任务。
- 🔥 **Google AI Studio**：**教学首选**。完全免费，用 Google 账号直接登录，无需信用卡。由 Google 出品，背后是 Gemini 模型。
- 🔥 **Bolt.new**：功能最完整的浏览器工具，支持全栈 App，免费额度慷慨。
- 🔥 **Claude Code**：命令行里的标杆，对大型代码库的理解能力最强。
- 🔥 **vibe-kanban**：特别的存在——一个专门管理 AI 代理的看板，可以同时跑 10+ 个 AI 代理并行工作。

> 各工具费用、免费额度及教学方案详见 [`tools-and-pricing.md`](./tools-and-pricing.md)

---

### 漏洞 4：Task Management 和 Documentation 工具是干什么的？

**答：它们是 Vibe Coding 的"操作系统"。**

光有 AI 工具还不够——当项目变大，你需要：

**① 任务管理工具（Task Management）**
让 AI 知道"整体目标是什么、现在做到哪里了、下一步是什么"

| 工具 | 干什么的 |
|------|---------|
| **Claude Task Master** | AI 自动把你的项目拆解成小任务，逐个执行，可用于 Cursor/Windsurf 等 |
| **vibe-kanban** | 可视化看板，同时编排 10+ 个 AI 代理，每个代理负责不同功能 |
| **CCPM** | 专门为 Claude Code 设计，用 GitHub Issues + Git worktree 管理并行代理 |
| **Boomerang Tasks** | Roo Code 的功能，自动把复杂任务分解成可执行的小块 |

**② 文档工具（Documentation for AI）**
让 AI 了解你的项目背景、规范和规则，避免它乱做决定

| 工具 | 干什么的 |
|------|---------|
| **AGENTS.md** | Linux 基金会主导的标准格式，一个文件告诉所有 AI 代理"这个项目的规则是什么" |
| **Context7** | 实时把最新的库文档注入 AI 的对话，避免 AI 用过时的 API |
| **.cursorrules** | Cursor 专用，告诉 AI "在这个项目里，写代码要遵循哪些规范" |
| **claude-reflect** | 自动学习你对 Claude 的纠正，把经验同步到 CLAUDE.md |

---

### 填充专业词汇

**Agentic 模式（智能体模式）是什么？**

- **普通 AI 助手模式**：你问 → 它答 → 你复制粘贴 → 你手动执行
- **Agentic 模式**：你给一个目标 → AI 自动规划步骤 → 自动读文件 → 自动写代码 → 自动运行 → 自动看报错 → 自动修复 → 反复循环直到完成

举个例子："给我的网站加一个登录功能"
- 非 Agentic：AI 给你一段代码，你自己决定怎么用
- Agentic（Cursor/Claude Code）：AI 读你整个项目 → 创建 login.js → 修改路由 → 更新数据库 schema → 运行测试 → 发现报错 → 自动修复 → 报告完成

---

**MCP（Model Context Protocol，模型上下文协议）是什么？**

想象 AI 是一个超级聪明但**被关在房间里**的人：它很聪明，但只能看你放进房间的东西。

**MCP 是给这个房间开窗户和门的标准**——让 AI 可以主动去拿它需要的东西：

```
没有 MCP:
你 → 手动复制 Figma 设计截图 → 粘贴给 AI → AI 才能看到设计
你 → 手动粘贴数据库结构 → AI 才能理解你的数据

有了 MCP:
AI → 直接连接 Figma → 自动读取你的设计稿
AI → 直接连接数据库 → 自动读取你的表结构
AI → 直接访问 GitHub → 自动读取 Issues 和 PR
```

MCP 就像 **USB-C 接口的标准化**——不同的工具（Figma、数据库、GitHub、浏览器）都可以用统一的方式接入 AI。

**支持 MCP 的工具**：Claude Code、Cline、Roo Code、Kilo Code、Continue、Cursor、GitHub Copilot 等。

---

**Git Worktree（Git 工作树）是什么？**

你的代码项目通常像一张桌子：只有一张，一次只能在上面做一件事。

**Git Worktree** 让你把同一个项目复制成多张独立的桌子，每张桌子可以同时工作，互不干扰。

在 Vibe Coding 里：
- 桌子 1：AI Agent A 在做"登录功能"
- 桌子 2：AI Agent B 在做"首页改版"  
- 桌子 3：AI Agent C 在修"购物车 bug"

三个 AI 同时工作，最后合并结果。**CCPM、Superset** 这类工具就是用这个原理实现多 AI 并行。

---

## Feynman Step 4：精炼解释
## Feynman Step 4: Refined Explanation

**最终简单解释：**

Vibe Coding 工具分 4 类：浏览器工具（最简单，面向所有人）、AI IDE（给开发者用的超级编辑器）、命令行工具（给工程师用的最强武器）、插件（给不想换工具的人）。

选哪个取决于你的技术背景和项目阶段：**不懂代码用浏览器工具，会代码用 Cursor，住在终端用 Claude Code**。

但工具只是冰山一角——背后的关键是 **Agentic 能力**（AI 能自主行动，不只是回答问题）和 **MCP**（让 AI 能连接 Figma、数据库、GitHub 等外部系统）。当项目变大，还需要**任务管理**和**文档工具**来告诉 AI 整个项目的背景和规则。

**精炼类比：**

Vibe Coding 工具就像**不同级别的餐厅：**

- 浏览器工具 = **外卖 App**：你在手机上点，餐厅做好送到你家，你什么都不用动手
- AI IDE = **有厨师的私厨**：厨师（AI）就在你家厨房，你告诉他想吃什么，他做，你可以随时说"再咸一点"
- CLI 工具 = **你自己是大厨**：你有一个极厉害的学徒（AI），它能做任何你说的事，但你得用专业语言指挥它
- 插件 = **给自家厨房装了高级设备**：你原来怎么做饭还怎么做，但多了很多智能辅助

**核心要点：**
1. 4 类工具对应 4 种使用场景，选对比选"最好的"更重要
2. Agentic 是关键——AI 能"自主行动"才是真正的 Vibe Coding
3. MCP 是 AI 的"感知器官"——让 AI 能看到、连接外部世界
4. 大项目需要任务管理 + 文档工具，否则 AI 会失去方向

> **30 秒版本：**
> "不懂代码用 Google AI Studio，在浏览器里描述你想要什么，10 分钟出原型。会写代码就用 Cursor，它能理解你整个项目、自动完成复杂任务。命令行爱好者用 Claude Code。不管用哪个，确保它支持 Agentic 模式——那才是真正的 Vibe Coding。"

---

## 🔬 课后练习
## 🔬 Hands-on Practice

### 练习 2-A：工具选择挑战（约 15 分钟）

读完下面 3 个场景，判断每个场景最适合用哪类工具，并说明理由：

**场景 1：** 小明是高中生，没学过编程，想做一个记录每天心情的日记 App，可以在手机上用。
- 他应该用哪类工具？为什么？

**场景 2：** 小红是前端工程师，公司项目在 VS Code 里，她想用 AI 帮她快速写重复性代码，但不想换编辑器。
- 她应该用哪类工具？为什么？

**场景 3：** 老张是创业者，有 10 年 Java 经验，想用 AI 重构一个 5 万行代码的老系统，需要同时处理多个模块。
- 他应该用哪类工具？为什么？

---

### 练习 2-B：上手 Cursor（如果你有编程基础，约 30 分钟）

1. 下载安装 [Cursor](https://cursor.com)（有免费额度）
2. 打开一个已有的项目，或新建一个空文件夹
3. 用 `Cmd+K`（Mac）或 `Ctrl+K`（Windows）唤出 AI
4. 让它帮你写一段简单代码，比如："写一个 Python 函数，输入一个列表，返回其中所有偶数"
5. 观察：它是直接给了答案，还是问了你问题？它会主动读其他文件吗？

---

### 练习 2-C：对比实验（约 30 分钟）

用**同一个提示词**在两个不同类型的工具里试验，记录结果：

**提示词：** *"做一个记账小工具：可以添加收入/支出记录（名称+金额+类别），显示总余额，用列表展示所有记录，可以删除记录。"*

- 工具 A：[Google AI Studio](https://aistudio.google.com)（浏览器工具）
- 工具 B：[v0.dev](https://v0.dev)（浏览器工具，偏 React 组件）

**比较维度：**
- ⏱️ 生成速度
- 🎨 界面美观程度
- 🛠️ 功能完整性
- 🔧 修改是否方便

---

### 练习 2-D：制作你的"工具选择卡片"

根据本讲内容，为自己制作一张个人化的工具选择参考卡：

```
我的背景：_______________
我主要的使用场景：_______________
我最适合的工具类型：_______________
我想先试试的具体工具：_______________
我还想进一步了解的概念：_______________
```

---

## 📚 延伸阅读
## 📚 Further Reading

| 资源 | 类型 | 难度 | 简介 |
|------|------|------|------|
| [Vibe Coding 101 with Replit - DeepLearning.AI](https://www.deeplearning.ai/short-courses/vibe-coding-101-with-replit/) | 课程 | ⭐ | 免费在线课，手把手带你用 Replit |
| [State of Vibe Coding Tools (2025)](https://www.linkedin.com/pulse/state-vibe-coding-tools-may-2025-nufar-gaspar-x1znf/) | 文章 | ⭐⭐ | 工具生态全景分析 |
| [AGENTS.md 官网](https://agents.md/) | 工具 | ⭐⭐ | 了解如何写 AI 代理指令文件 |
| [The Prompt Engineering Playbook](https://addyo.substack.com/p/the-prompt-engineering-playbook-for) | 文章 | ⭐⭐ | 面向程序员的提示词工程实战 |
| [AI Coding Guide (automata)](https://github.com/automata/aicodeguide) | 文档 | ⭐⭐ | 用 AI 开始编程的路线图 |

---

> **下一讲预告 →** Lecture 3：如何 Vibe Code 得好——光有工具不够，提示词技巧、工作流设计、文档规范，这些才是让 AI 真正帮到你的关键。
