# cc-sdd-flow

Specification Driven Development (SDD) 工作流框架，为 AI 辅助软件开发提供结构化的规格驱动开发方法论。

## 什么是 SDD

SDD 是一种以规格说明书为驱动的开发方法，强制 AI Agent 在写代码之前先完成业务规格 → 技术方案 → 任务分解三个阶段，避免"边想边写"导致的方向偏差和过度设计。

核心原则（定义在 `meta/constitution.md`）：
- **简单性**：只实现 spec 要求的功能，YAGNI
- **测试先行**：Red → Green → Refactor，不可跳步
- **明确性**：显式错误处理，无隐式全局状态
- **单一职责**：每个模块只做一件事

## 工作流

```
/init_project        →  初始化项目骨架
/generate_spec       →  业务规格说明书 (spec.md)
/generate_plan       →  技术实现方案   (plan.md)   ← 含 POC 验证步骤
/generate_task       →  原子化任务列表 (tasks.md)
```

每个功能模块的产出物存放在 `specs/{module}/` 目录下：

```
specs/
  001-feature-name/
    spec.md      # 业务规格（What/Why）
    plan.md      # 技术方案（How）
    tasks.md     # 任务清单（TDD 顺序）
    poc/         # POC 验证结果（按需）
```

## 仓库结构

```
commands/          # 命令定义（供 Claude Code 执行）
meta/              # 核心方法论文档
  constitution.md  # 开发原则（最高权威）
  CLAUDE.md        # SDD 工作流说明（供目标项目参考）
  AGENT_SAMPLE.md  # 技术标准示例
rules/
  testing.md       # 测试分层规范
templates/         # spec / plan / task 文档模板
```

## 快速开始

将本仓库的 `commands/`、`meta/`、`rules/`、`templates/` 复制到目标项目的 `.claude/` 目录下，然后在 Claude Code 中运行：

```
/init_project <project_name>
```

之后按需求描述依次运行 `/generate_spec` → `/generate_plan` → `/generate_task`，再按 `tasks.md` 逐任务执行。
