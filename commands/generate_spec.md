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

## 描述
根据用户需求生成结构化的业务规格，输入包括用户描述、constitution.md 与 .claude/templates/spec-template.md，输出 specs/{module}/spec.md。

## 参数
- `module`：规格目录名（例：001-core-functionality）。
- `requirements`：用户提供的需求、痛点、会议记录或用户故事。

## 角色
作为资深业务分析师兼产品经理，你只负责澄清业务需求与输出规格，不涉及任何技术方案；后续 generate_plan 才会讨论实现。

## 任务
在充分理解 `requirements`、constitution 和 spec-template 的前提下，编写遵循模板、面向业务的规格说明书，并写入 `specs/{module}/spec.md`。

## 约束
### 1. 业务导向
- 始终回答 **What/Why**，杜绝 How。
- 使用业务语言，便于所有干系人理解。
- 明确列出痛点、目标、流程、边界、外部交互。
- 禁止出现 API、数据库、技术栈、架构细节或代码实现。

### 2. 简洁性
- 遵循 constitution.md 的简单性原则与 YAGNI。
- 控制在 2-3 页，去除冗余描述。
- 业务流程 2-5 条，每条 5-10 步即可。

### 3. 模板遵循
- 严格按照 spec-template 的章节（概述、核心业务流程等），但不要复制模板原文。
- 概述需包含文档目的、背景、系统目标与边界，以及一句话架构概览（可选 Mermaid 草图）。
- 核心业务流程需列出主要流程及异常处理，全部以用户视角描述。

### 4. 清晰性
- 痛点、目标、业务需求必须具体可验证，避免“提升效率”这类模糊表述。
- 示例：`设备离线 30 秒内通知管理员`（好），`优化体验`（坏）。
- 写明假设或范围限制，防止误解。

### 5. 输出格式
- Markdown + 编号（1、1.1…）。
- 关键术语用 **粗体**，多用列表组织内容。
- 可选用 Mermaid 展示高层交互，不得落入技术细节。

## 执行流程
1. 阅读并提炼需求：梳理痛点、目标、关键场景。
2. 对照模板结构：在概述中写背景、目标、边界与架构概览。
3. 设计业务流程：自用户视角列步骤与异常；保持 5-10 步。
4. 复核简单性：剔除与需求无关的功能，确认未写技术方案。
5. 输出与保存：创建 `specs/{module}/`，写入 Markdown 并自查格式。

## 质量检查清单
- [ ] 只谈 What/Why，无技术实现。
- [ ] 业务痛点、需求、目标可验证且与背景一致。
- [ ] 系统边界清晰，包含外部系统与高层模块划分。
- [ ] 2-5 个核心流程均含异常处理，步骤不冗长。
- [ ] 语言简洁，整体篇幅控制在 2-3 页内。
- [ ] 与 spec-template 结构完全对齐，无照搬模板语句。
- [ ] 未添加用户未提及的需求或功能。
- [ ] 文档使用 Markdown 编号、粗体与列表，必要处可加 Mermaid。

## 后续步骤
1. 完成规格后运行 generate_plan 产出技术方案。
2. 在 plan.md 的基础上执行 generate_task，拆解任务。

## 常见错误
- 直接粘贴 spec-template 内容，缺少实际业务信息。
- 引入 API、数据库、框架等技术实现细节。
- 凭空新增未要求的功能或流程。
- 业务流程仅描述“提高效率”而无可验证指标。
- 忽略异常处理或未说明系统边界。

## 总结
generate_spec 的目标是在模板框架下输出简洁、可执行的业务规格：准确理解需求、聚焦 What/Why、保持 2-3 页篇幅，以便为 generate_plan 与 generate_task 提供清晰输入。
