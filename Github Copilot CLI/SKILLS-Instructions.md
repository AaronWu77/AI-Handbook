# GitHub Copilot CLI - SKILLS 详细指南

## 目录
1. [SKILL 的基本原理](#skill-的基本原理)
2. [SKILL 在 Copilot CLI 中的使用方法](#skill-在-copilot-cli-中的使用方法)
3. [SKILL 结构规范](#skill-结构规范)
4. [实践示例](#实践示例)

---

## SKILL 的基本原理

### 什么是 SKILL？

**SKILL** 是 GitHub Copilot CLI 中的**特殊能力扩展**，用于为特定工作流程或问题领域提供专门的上下文和工具集合。SKILL 本质上是一个**自包含的指令集**，包含：

- **明确的适用场景**（什么时候触发）
- **专门的行为逻辑**（怎样工作）
- **支持资源**（参考文档、数据、模板等）

### SKILL 的核心特征

| 特征 | 说明 |
|------|------|
| **专一性** | 每个 SKILL 只做一件事，而且做得很好 |
| **自包含性** | SKILL 包含执行所需的所有指令和资源 |
| **触发机制** | 通过特定关键词、问题或上下文激活 |
| **工具调用** | SKILL 调用 CLI 工具（如 `ask_user`、`sql`、`view`、`bash` 等） |
| **状态管理** | 某些 SKILL（如 cli-mastery）可维护进度追踪数据库 |
| **模块化** | SKILL 之间相对独立，可并行或按需加载 |

### SKILL 的工作流程

```
用户输入
    ↓
CLI 解析关键词/意图
    ↓
匹配对应 SKILL 的触发条件
    ↓
加载 SKILL 指令 (SKILL.md)
    ↓
执行 SKILL 定义的逻辑
    ├─ 读取支持资源（references/）
    ├─ 调用工具（ask_user, sql, view, bash 等）
    ├─ 处理用户交互
    └─ 维护状态（如需要）
    ↓
返回结果给用户
```

### SKILL 的生命周期

#### 1. **初始化阶段**
- SKILL 检测是否首次被调用
- 创建必要的数据表或配置（如 `cli-mastery` 的 `mastery_progress` 表）
- 初始化状态变量

#### 2. **活跃阶段**
- SKILL 根据用户输入执行特定逻辑
- 与用户通过工具交互（如 `ask_user` 提问）
- 读取和更新支持资源
- 维护进度或状态

#### 3. **完成/退出阶段**
- SKILL 提供结果或反馈
- 更新持久化数据（数据库、文件）
- 向用户提示下一步行动

---

## SKILL 在 Copilot CLI 中的使用方法

### 1. 调用 SKILL 的方式

#### 方式 A：通过触发词/关键词
每个 SKILL 都有预定义的**触发短语**。在 CLI 中输入这些短语会自动激活对应 SKILL：

**示例：**
```bash
# 激活 cli-mastery SKILL
> cliexpert
> teach me the Copilot CLI
> quiz me on slash commands
> CLI final exam

# 激活 create-readme SKILL
> create readme
> generate README

# 激活 exam-ready SKILL
> I have an exam tomorrow on mathematics
> explain [topic] from my notes
> I only have 2 hours, help me prepare

# 激活 suggest-awesome-github-copilot-skills SKILL
> suggest skills for my project
> find relevant Copilot skills
```

#### 方式 B：显式技能调用（如果支持）
某些 CLI 版本可能支持显式指令：
```bash
> /skill cli-mastery
> /skill create-readme
```

### 2. 与 SKILL 交互的模式

#### 模式 1：指导学习（cli-mastery）
```
用户: cliexpert
  ↓
CLI: 加载第一个模块，教授斜杠命令
  ↓
用户: 完成学习，选择"quiz me"
  ↓
CLI: 呈现 5+ 道测试题
  ↓
用户: 回答问题
  ↓
CLI: 计算 XP，更新等级，提供反馈
```

**XP 奖励系统：**
- 完成课程：+20 XP
- 正确答题：+15 XP
- 完美测验（全对）：+50 XP
- 完成场景挑战：+30 XP

**等级进度：**
```
0     → Newcomer
100   → Apprentice
250   → Navigator
400   → Practitioner
550   → Specialist
700   → Expert
850   → Virtuoso
1000  → Architect
1150  → Grandmaster
1500  → Wizard
```

#### 模式 2：内容生成（create-readme）
```
用户: create readme
  ↓
CLI: 分析整个项目结构
  ↓
CLI: 生成专业、清晰、结构完善的 README.md
  ↓
用户: 获得可直接使用的 README 文件
```

**生成规则：**
- 遵循开源项目最佳实践
- 使用 GFM（GitHub Flavored Markdown）语法
- 不过度使用 emoji
- 不包含 LICENSE、CONTRIBUTING、CHANGELOG 等（这些有专门文件）

#### 模式 3：考试准备（exam-ready）
```
用户: 上传 PDF 或粘贴笔记 + 提供考试大纲
  ↓
CLI: 逐项处理每个大纲主题
  ↓
CLI: 从材料中提取：
    - 定义
    - 关键点 (3-5 个)
    - 重点关键词
    - 图解描述
    - 考试可用的答题句式
    - 自测题
  ↓
用户: 获得考试备考手册
```

**关键约束：**
- ⚠️ **严格限制**：只从提供的材料中提取，不添加外部知识
- 考试类型支持：MCQ（单选）、简答题、长答题
- 时间约束支持：如用户说"我只有 2 小时"，CLI 将按优先级压缩内容

#### 模式 4：技能推荐（suggest-awesome-github-copilot-skills）
```
用户: suggest skills for my project
  ↓
CLI: 分析仓库上下文（语言、框架、项目类型）
  ↓
CLI: 查询 awesome-copilot 官方技能库
  ↓
CLI: 对比本地已有技能，识别缺失和过时技能
  ↓
CLI: 生成推荐表格，显示：
    - 推荐的技能
    - 为什么推荐
    - 是否已安装 / 是否过时
    - 官方链接
  ↓
用户: （确认后）自动下载/更新技能
```

### 3. 常见交互流程

#### 流程 1：学习并测验（cli-mastery）
```bash
1. > cliexpert
   👉 开始第一模块：斜杠命令

2. > quiz me
   👉 生成 5+ 测试题

3. 用户逐个回答...

4. > scenario challenge
   👉 加载场景挑战题

5. > 继续学习下一模块或参加最终考试
```

#### 流程 2：准备考试（exam-ready）
```bash
1. > I have an exam on Biology tomorrow, 3 hours preparation

2. 👉 CLI 提示："请上传你的笔记 PDF 或粘贴文本"

3. 用户提供笔记和大纲...

4. 👉 CLI 生成按优先级排序的备考文档
   - 定义
   - 关键点
   - 关键词
   - 图解
   - 准备好的答题句式
   - 自测题

5. 用户复习或继续：> quiz me on [topic]
```

#### 流程 3：获得技能建议（suggest-awesome-github-copilot-skills）
```bash
1. > suggest skills for my TypeScript React project

2. 👉 CLI 分析：
   - 文件类型 (.ts, .tsx)
   - 框架 (React)
   - 项目结构

3. 👉 CLI 查询 awesome-copilot，生成表格：
   | 技能名称 | 描述 | 已安装 | 过时 | 推荐理由 |
   | react-best-practices | React 最佳实践... | ✅ | ❌ | 覆盖现有能力 |
   | ts-type-checking | TypeScript 类型检查... | ❌ | - | 增强类型安全性 |

4. > 安装 ts-type-checking
   👉 CLI 自动下载到 .github/skills/ 目录
```

### 4. SKILL 的数据持久化

某些 SKILL（如 **cli-mastery**）维护进度数据：

```sql
-- cli-mastery 创建的表
CREATE TABLE mastery_progress (
    key TEXT PRIMARY KEY,      -- 'xp', 'level', 'module'
    value TEXT                 -- XP 值、当前等级、模块编号
);

CREATE TABLE mastery_completed (
    module TEXT PRIMARY KEY,   -- 完成的模块
    completed_at TEXT          -- 完成时间戳
);
```

**跨会话特性：** 用户的学习进度在不同会话中保持，XP 和等级会累积。

---

## SKILL 结构规范

### 文件夹结构

```
SKILLS/
├── cli-mastery/
│   ├── SKILL.md                    # 主指令文件
│   └── references/                 # 支持资源
│       ├── module-1-slash-commands.md
│       ├── module-2-keyboard-shortcuts.md
│       ├── ... (module 3-8)
│       ├── scenarios.md
│       └── final-exam.md
│
├── create-readme/
│   └── SKILL.md                    # 简单工具，无额外资源
│
├── exam-ready/
│   └── SKILL.md
│
└── suggest-awesome-github-copilot-skills/
    └── SKILL.md
```

### SKILL.md 前言格式

```markdown
---
name: skill-name                    # 技能名称（kebab-case）
description: 'Short description'    # 简短描述
metadata:
  version: 1.0.0                    # 版本
license: MIT                        # 许可证（可选）
---

# Skill 标题

正文...
```

### 必要的 SKILL.md 内容部分

| 部分 | 说明 |
|------|------|
| **前言** | YAML 格式的元数据 |
| **角色/定义** | 这个 SKILL 是什么，做什么 |
| **触发条件** | 什么时候激活（关键词、短语） |
| **工具调用** | 使用哪些 CLI 工具 |
| **行为逻辑** | 详细的执行步骤 |
| **规则/约束** | 不能做什么、必须做什么 |
| **输出格式** | 返回给用户的结果格式 |

---

## 实践示例

### 示例 1：使用 cli-mastery 学习

```bash
# 会话 1：开始学习
$ copilot
> cliexpert

输出：
  欢迎来到 Copilot CLI 掌握课程！
  
  Module 1: Slash Commands
  ════════════════════════
  
  斜杠命令是快速访问特定 CLI 功能的快捷方式...
  
  [详细讲解...]
  
  现在你获得了 +20 XP。当前等级: Newcomer (20/100)
  
  下一步？
  - 输入 "quiz me" 来测试理解
  - 输入 "cliexpert" 继续下一模块
  - 输入 "reference" 查看总结

> quiz me

输出：
  ⏱️ 5 题快速测验 - Slash Commands
  
  Question 1: 哪个斜杠命令用于打开 Copilot 空间？
  A) /space
  B) /copilot
  C) /help
  
  [选择...]

> A

输出：
  ✅ 正确！+15 XP
  当前 XP: 35/100
  
  [继续下一题...]

# 会话 2：继续学习
$ copilot
> cliexpert

输出：
  欢迎回来！你现在是 Apprentice 等级 (35/100 XP)
  
  进度：已完成 Module 1，可继续 Module 2...
```

### 示例 2：使用 exam-ready 准备考试

```bash
$ copilot
> I have an exam on World History tomorrow, 4 hours to study

输出：
  准备好了！请提供你的学习材料。
  
  请粘贴你的笔记或上传 PDF，然后列出考试大纲的主题。

> [用户粘贴 5 页笔记和大纲...]

输出：
  分析完成！根据你的 4 小时准备时间，这是优先级排序：
  
  优先级 1 (70% 学习时间)：
  ─────────────────
  - 世界大战 I（笔记中占 40%）
  - 冷战（笔记中占 25%）
  
  优先级 2 (25% 学习时间)：
  ─────────────────
  - 启蒙运动
  - 工业革命
  
  优先级 3 (5% 学习时间)：
  ─────────────────
  - 古代文明概述
  
  准备好详细信息？输入：
  - "explain WWI" — 详细讲解第一次世界大战
  - "quiz me" — 自测题
  - "all" — 全部生成

> explain WWI

输出：
  ### 世界大战 I
  
  **定义：** 1914-1918 年的全球军事冲突，主要参战者分为协约国和同盟国...
  
  **关键点：**
  - 奥匈帝国皇储被刺杀引发...
  - 帝国主义竞争加剧...
  - 协约国最终胜利...
  
  **关键词要使用：** 协约国、帝国主义、塹壕战、一战
  
  **考试可以这样写：**
  "第一次世界大战（1914-1918）是由奥匈帝国皇储遇刺引发的，
   反映了帝国主义列强争夺全球霸权的冲突。协约国最终击败同盟国..."
  
  **自测题：**
  什么事件直接触发了第一次世界大战的开始？
  （答案：奥匈帝国皇储弗朗茨·斐迪南遇刺）
```

### 示例 3：使用 suggest-awesome-github-copilot-skills 获得建议

```bash
$ copilot
> suggest skills for my project

输出：
  分析中...你的项目：
  - 语言: TypeScript (70%), Python (20%)
  - 框架: React, FastAPI
  - 项目类型: 全栈 Web 应用
  
  ┌─────────────────────────────────────────────────────────────┐
  │ 推荐的 GitHub Copilot Skills                                 │
  └─────────────────────────────────────────────────────────────┘
  
  | Skill 名称           | 描述                | 已安装 | 推荐理由           |
  |─────────────────────|────────────────────|────────|────────────────────|
  | react-performance   | React 性能优化      | ❌    | 你的应用需要提速  |
  | ts-strict-mode      | TypeScript 严格模式 | ✅    | 已覆盖            |
  | fastapi-best-prac   | FastAPI 最佳实践    | ⚠️    | 已安装但需更新    |
  | docker-deployment   | Docker 部署         | ❌    | 完整化部署流程    |
  
  要安装 react-performance？输入 "install react-performance"
  要更新 fastapi-best-prac？输入 "update fastapi-best-prac"

> install react-performance

输出：
  下载中...
  ✅ react-performance 已安装到 .github/skills/
  
  你现在可以使用这个 skill：输入相关问题，Copilot 会调用这个 skill。
```

---

## 总结

| 概念 | 说明 |
|------|------|
| **SKILL 是什么** | CLI 的专门能力扩展，自包含指令集 |
| **怎样触发** | 通过关键词、短语或显式命令 |
| **怎样交互** | 基于 SKILL 类型的不同模式（学习、生成、推荐等） |
| **数据持久化** | 某些 SKILL 维护进度库，跨会话保留状态 |
| **文件结构** | SKILL.md + 可选的 references/ 支持资源 |
| **核心特征** | 专一、自包含、模块化、工具调用、状态管理 |

通过 SKILL，Copilot CLI 从一个通用助手演进为**多个专业领域的专家**，每个 SKILL 都专注于解决特定类别的问题。
