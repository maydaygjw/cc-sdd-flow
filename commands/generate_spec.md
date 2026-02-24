---
name: generate_spec
description: 根据用户需求描述生成结构化的业务规格说明书
parameters:
  - name: module
    description: 规格说明模块目录名称（例如：001-core-functionality）
    required: true
  - name: requirements
    description: 用户的需求描述（自然语言、业务痛点、用户故事等）
    required: true
inputs:
  - constitution.md
  - .claude/templates/spec-template.md
outputs:
  - specs/{module}/spec.md
---

# 命令：generate_spec

## 职责边界

你的职责是把用户的模糊需求转化为可驱动开发的业务规格。**不做技术决策，不补充用户未提及的功能。** 技术方案由后续的 `generate_plan` 负责。

## 前置检查

执行前先确认以下文件存在，缺失则提示用户创建后再继续：
- `constitution.md`
- `.claude/templates/spec-template.md`

## 执行流程

### 第一步：理解需求，输出确认摘要

阅读 `$requirements`，提炼以下内容并**输出给用户确认**，等待用户确认后再继续：

```
我理解的需求如下，请确认：
- 核心目标：[一句话]
- 主要场景：[2-3 条]
- 明确不包含：[边界排除]
- 待澄清的问题：[如有疑问列出，没有则省略]
```

### 第二步：生成 spec.md

用户确认后，按照 `spec-template.md` 的章节结构生成规格，写入 `specs/{module}/spec.md`。

生成时遵循以下原则：

**只谈 What/Why，不谈 How**
- 禁止出现：API、数据库、技术栈、框架、架构图、代码
- 允许出现：业务流程、用户操作、系统行为、验收条件

**具体可验证，不写模糊描述**
- 示例见 spec-template.md 的注释

**只写用户提到的内容**
- 不补充用户未提及的功能
- 不添加"后续可扩展"的功能

**信息密度优先，不设硬性页数**
- 去除冗余描述，避免同一内容在多个章节重复出现
- 概述、目标、背景不要分成三个重复的小节

### 第三步：自查

写完后对照检查清单自查，发现问题自行修正，无需输出检查过程。

## 检查清单

- [ ] 只谈 What/Why，无技术实现细节
- [ ] 业务目标具体可验证，无模糊表述
- [ ] 系统边界清晰，外部依赖和内部模块均已列出
- [ ] 每个业务流程均包含异常处理
- [ ] 非功能需求只保留影响验收或实现决策的内容（通用性能指标不写）
- [ ] 验收标准可直接用于测试，无主观判断项
- [ ] 未添加用户未提及的功能
- [ ] 章节结构与 spec-template 对齐，无照搬模板原文

## 常见错误

- 直接复制 spec-template 的占位文字而不填入实际内容
- 在概述、目标、背景三节中重复描述同一件事
- 写了"技术挑战"或"系统架构"——这些属于 plan.md
- 非功能需求填了一堆无法验证的通用指标
- 跳过第一步确认直接生成，导致方向偏差

## 后续步骤

spec.md 确认后：
1. 运行 `generate_plan` 生成技术方案（plan.md）
2. 运行 `generate_task` 拆解任务清单（tasks.md）