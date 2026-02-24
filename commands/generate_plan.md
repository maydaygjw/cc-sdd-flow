---
name: generate_plan
description: 根据规格说明书生成技术实现方案
parameters:
  - name: module
    description: 规格说明模块目录名称（例如：001-core-functionality）
    required: true
inputs:
  - specs/{module}/spec.md
  - constitution.md
  - .claude/templates/plan-template.md
outputs:
  - specs/{module}/plan.md
---

# 命令：generate_plan

## 职责边界

将 spec.md 中的业务需求转化为可直接指导开发的技术方案。**只做技术决策，不重复业务描述，不补充 spec 未提及的功能。**

## 前置检查

执行前确认以下文件存在，缺失则提示用户后停止：
- `specs/{module}/spec.md`
- `constitution.md`
- `.claude/templates/plan-template.md`

## 执行流程

### 第一步：理解输入，输出确认摘要

阅读 spec.md 和 constitution.md，提炼以下内容并**输出给用户确认**，等待确认后再继续：

```
我理解的技术方案范围如下，请确认：
- 需要新增/修改的模块：[列出]
- 关键技术决策点：[列出需要选型的地方]
- 明确不需要的章节：[如 API 设计、数据库设计等不适用的内容]
- 待澄清的问题：[如有则列出，没有则省略]
```

### 第二步：生成 plan.md

用户确认后，按照 plan-template.md 的章节结构生成方案，写入 `specs/{module}/plan.md`。

生成时遵循以下原则：

**按需取用模板章节**
- plan-template 是菜单，不是必填表
- 没有数据库就不写数据库设计，没有 API 就不写 API 设计
- 项目类型决定需要哪些章节：CLI 工具关注模块设计和资源生命周期，Web 服务关注接口和数据层，脚本工具可能只需要技术选型和流程

**技术选型必须有理由**
- 每个选型对应 spec 中的具体需求或约束
- 禁止无理由的选型，如"使用 PostgreSQL"而不说明为什么

**不重复 spec 的内容**
- 业务流程已在 spec.md 中描述，plan 只写技术实现细节
- 关键流程章节只写有技术复杂度的部分（组件协作、数据流转、异常路径）

**纯函数显式标注**
- 模块设计中明确标出哪些是纯函数（无副作用、可单元测试）
- 这直接影响 generate_task 的测试任务拆分

**风险只写真实的**
- 不填通用风险（"网络可能不稳定"、"磁盘可能满"）
- 只写该方案特有的、影响实现决策的风险

### 第三步：自查

写完后对照检查清单自查，发现问题自行修正，无需输出检查过程。

## 检查清单

- [ ] 所有技术选型都有对应 spec 需求的理由
- [ ] 未包含 spec 中没有的功能或模块
- [ ] 模块设计中纯函数已显式标注
- [ ] 关键流程只写技术复杂度高的部分，未重复 spec 的业务描述
- [ ] 测试策略与 constitution.md 的测试层次对齐
- [ ] 风险只列该方案特有的真实风险
- [ ] 合宪性审查已完成，不符合项有说明
- [ ] 不适用的模板章节已跳过，未强行填充

## 常见错误

- 照抄 spec 的业务流程描述到 plan 里
- 强行填写不适用的章节（如 CLI 工具写 API 设计）
- 技术选型没有理由，或理由是"业界常用"
- 风险填了一堆通用项，无实际参考价值
- 跳过第一步确认直接生成，导致方向偏差
- 引用了项目中不存在的文件（如 AGENTS.md）

## 后续步骤

plan.md 确认后，运行 `generate_task` 拆解任务清单（tasks.md）