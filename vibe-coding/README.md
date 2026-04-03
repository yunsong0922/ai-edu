# Vibe Coding 课程 — 完整讲义
# Vibe Coding Course — Complete Lecture Notes

> 基于 [awesome-vibe-coding](https://github.com/filipecalegario/awesome-vibe-coding) 资源整理
> 采用 Feynman 教学法：简单解释 → 找漏洞 → 填漏洞 → 精炼解释
> 面向：中学生 / 初学者
> 语言：中英双语

---

## 课程结构 / Course Structure

```
Vibe Coding 课程（共 4 讲 + 1 个综合项目）
│
├── 📘 Lecture 1: 什么是 Vibe Coding？
│   ├── 核心概念与起源（Karpathy 2025）
│   ├── Vibe Coding 光谱：从零代码到 AI 辅助专业开发
│   ├── AI 写代码的原理（LLM 的本质）
│   ├── 能做什么 vs 不能做什么
│   └── 🔬 练习：体验 Google AI Studio 纯 Vibe Coding
│
├── 🗺️ Lecture 2: 工具全景图
│   ├── 4 大工具类别：浏览器/AI IDE/CLI/插件
│   ├── 工具选择决策树
│   ├── 关键概念：Agentic、MCP、Git Worktree
│   ├── 支撑层：任务管理 + 文档工具
│   └── 🔬 练习：工具对比实验
│
├── ⚙️ Lecture 3: 如何 Vibe Code 得好
│   ├── 好的提示词原则（4 个核心原则）
│   ├── 专业工作流（5 阶段）
│   ├── 文档工具：AGENTS.md / CLAUDE.md / Context7
│   ├── 上下文窗口问题与解决方案
│   └── 🔬 练习：完整工作流实战
│
├── ⚠️ Lecture 4: 风险、边界与未来
│   ├── 安全漏洞：SQL注入、密码存储、权限验证
│   ├── 技术债：为什么 AI 代码容易变成烂摊子
│   ├── 开源危机：AI 代码如何影响 GitHub 生态
│   ├── Amazon 生产事故案例分析
│   ├── 对学生的建议：机会 vs 陷阱
│   └── 🔬 练习：安全审计实验 + 辩论
│
└── 🚀 Final Project: 从零构建真实产品
    ├── 6 阶段完整工作流
    ├── 4 个可选项目（班级公告板/投票/单词卡/自选）
    ├── 每阶段交付物要求
    ├── 安全审查模板
    └── 反思报告框架
```

---

## 学习路径 / Learning Path

```
Week 1          Week 2          Week 3          Week 4
────────        ────────        ────────        ────────
Lecture 1       Lecture 2       Lecture 3       Lecture 4
什么是           工具全景图       如何做得好       风险与未来
Vibe Coding     
                                Week 5-6
                                ────────
                                Final Project
                                综合实战
```

---

## 核心知识图谱 / Knowledge Map

```
                    VIBE CODING
                         │
        ┌────────────────┼────────────────┐
        │                │                │
    是什么？           用什么？          怎么做好？
    (Lecture 1)      (Lecture 2)       (Lecture 3)
        │                │                │
   LLM 原理          4大工具类别        提示词原则
   光谱概念          Agentic/MCP       5阶段工作流
   能/不能           任务管理          AGENTS.md
                     文档工具          git commit
                         │
                    有什么风险？
                    (Lecture 4)
                         │
                   安全漏洞
                   技术债
                   开源影响
                   职业思考
                         │
                    Final Project
                    综合实战验证
```

---

## 关键原则速查 / Key Principles Quick Reference

### 提示词原则
1. **具体** — 描述结果，不描述步骤
2. **有背景** — 每次告诉 AI 项目是什么
3. **一次一件事** — 不贪多
4. **会迭代** — 第一次不完美很正常

### 工作流原则
1. **先规划** — 拆成原子任务再动手
2. **频繁 commit** — 每完成一个功能就 `git commit`
3. **用 AGENTS.md** — 让 AI 始终了解项目规则
4. **新对话做新功能** — 不要在一个超长对话里做所有事

### 安全原则
1. **登录/支付/用户数据** — 必须人工审查
2. **让 AI 审查自己的代码** — 找安全漏洞
3. **不信任，要验证** — AI 代码 ≠ 安全代码

---

## 工具速查表 / Tools Quick Reference

| 你是谁 | 推荐工具 | 链接 |
|-------|---------|------|
| 完全不懂代码 | Google AI Studio | aistudio.google.com |
| 想快速做原型 | Bolt.new | bolt.new |
| React 开发者 | v0 by Vercel | v0.dev |
| 会写代码的开发者 | Cursor | cursor.com |
| 住在终端的工程师 | Claude Code | claude.ai |
| 不想换编辑器 | GitHub Copilot / Cline | copilot.github.com |
| 管理多个 AI 代理 | vibe-kanban | github.com/BloopAI/vibe-kanban |

> 费用、免费额度及教学方案对比见 [`tools-and-pricing.md`](./tools-and-pricing.md)

---

## 文件列表 / File Index

| 文件 | 内容 | 建议学习时长 |
|------|------|------------|
| `lecture-01-what-is-vibe-coding.md` | 概念、原理、光谱 | 1.5 小时（含练习）|
| `lecture-02-tools-landscape.md` | 工具分类、选择指南、MCP | 2 小时（含练习）|
| `lecture-03-how-to-vibe-code-well.md` | 提示词、工作流、文档 | 2.5 小时（含练习）|
| `lecture-04-risks-and-future.md` | 安全、技术债、未来 | 2 小时（含练习）|
| `final-project.md` | 综合实战项目 | 4-6 小时 |

**总学习时长估计：约 12-14 小时**

---

*课程基于 [filipecalegario/awesome-vibe-coding](https://github.com/filipecalegario/awesome-vibe-coding) 整理，遵循 CC0 协议。*
