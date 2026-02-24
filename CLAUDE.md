# CLAUDE.md

> **最高优先级**：核心开发原则在 `meta/constitution.md` 中定义，效力高于本文件。

## 项目概述

**cc-sdd-flow** 是一个 Specification Driven Development (SDD) 工作流框架，为 AI 辅助软件开发提供模板、命令和规范，引导 AI Agent 以规格驱动的方式完成软件开发。

## 仓库结构

```
/
├── commands/                  # AI Agent 可执行的命令定义
│   ├── init_project.md        # 初始化 Python SDD 项目结构
│   ├── generate_spec.md       # 根据需求描述生成业务规格说明书
│   ├── generate_plan.md       # 根据规格说明书生成技术实现方案
│   ├── run_poc.md             # 验证技术假设，输出可行性结论
│   └── generate_task.md       # 根据技术方案生成原子化任务列表
├── meta/                      # 核心方法论文档
│   ├── constitution.md        # 开发原则（最高权威）
│   ├── CLAUDE.md              # SDD 工作流方法论（供目标项目参考）
│   └── AGENT_SAMPLE.md        # 后端/前端技术标准示例
├── rules/
│   └── testing.md             # 测试分层规范（单元/集成/E2E）
└── templates/                 # 文档模板
    ├── spec-template.md       # 业务规格说明书模板
    ├── plan-template.md       # 技术方案模板
    └── task-template.md       # 任务分解模板
```

## SDD 工作流（4 个阶段）

### 阶段 0：项目初始化
```
/init_project <project_name> <python_version>
```
为空项目建立 SDD 骨架：venv、目录结构、CLAUDE.md 框架、constitution.md。

### 阶段 1：业务规格（spec.md）
```
/generate_spec <module> "<需求描述>"
```
- 输入：用户需求描述（自然语言）
- 输出：`specs/{module}/spec.md`
- 原则：只谈 What/Why，不谈 How，不补充用户未提及的功能

### 阶段 2：技术方案（plan.md）
```
/generate_plan <module>
```
- 输入：`specs/{module}/spec.md`、`constitution.md`、`templates/plan-template.md`
- 输出：`specs/{module}/plan.md`
- 含 POC 步骤：遇到复杂技术决策点时，先运行 `run_poc` 验证后再生成 plan

#### POC 验证（按需）
```
/run_poc <module> "<技术假设>"
```
- 以子代理方式运行，避免污染主对话上下文
- 输出：`specs/{module}/poc/{slug}/result.md`
- 结论：可行 / 不可行 / 有条件可行

### 阶段 3：任务分解（tasks.md）
```
/generate_task <module>
```
- 输入：`specs/{module}/spec.md`、`specs/{module}/plan.md`、`constitution.md`、`templates/task-template.md`
- 输出：`specs/{module}/tasks.md`
- 原则：每个任务只动一个文件，TDD 顺序（测试在实现之前）

## 核心约束

**对所有命令通用：**
- 每个命令执行前必须输出确认摘要，等待用户确认后再继续
- 只处理输入文件中已有的内容，不补充未提及的功能
- 按需取用模板章节，不强行填充不适用的内容

**任务原子性（generate_task）：**
- 一个任务 = 一个文件的新建或修改
- 依赖关系：`depends: [task_ids]`
- 可并行任务标记 `[P]`
- 顺序：基础设施 → 核心模块 → 集成层 → E2E

**TDD 铁律（不可协商）：**
- Red → Green → Refactor，不可跳步
- 测试任务必须在实现任务之前
- 测试分层规范见 `rules/testing.md`

## 治理

`meta/constitution.md` 效力高于：
- 本 CLAUDE.md
- meta/AGENT_SAMPLE.md
- 任何会话内指令

所有 plan 和 tasks 生成前必须通过合宪性审查。
