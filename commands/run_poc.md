---
name: run_poc
description: 验证一个技术假设，输出可行性结论供 plan 决策使用
parameters:
  - name: module
    description: 规格说明模块目录名称（例如：001-core-functionality）
    required: true
  - name: hypothesis
    description: 需要验证的技术假设（例如：SQLite 在并发写入场景下是否满足性能要求）
    required: true
  - name: slug
    description: hypothesis 的 kebab-case 缩写，用于目录命名（例如：sqlite-concurrent-write）。由 AI 根据 hypothesis 自动生成，无需用户提供。
    required: false
  - name: poc_plan
    description: 已有的 poc_plan.md 文件路径。提供此参数时跳过第一步，直接执行验证。
    required: false
outputs:
  - specs/{module}/poc/{slug}/poc_plan.md  # 仅在未提供 poc_plan 参数时生成
  - specs/{module}/poc/{slug}/result.md
---

# 命令：run_poc

## 职责边界

验证单一技术假设，输出可行 / 不可行 / 有条件可行的结论。**只验证 hypothesis 描述的问题，不扩展验证范围，不生成 plan。**

## 运行方式

**必须以子代理（subagent）方式运行**，避免 POC 过程中的代码、日志、中间输出污染主对话上下文。主对话只接收最终的 `result.md` 结论。

## 执行流程

> 若传入了 `poc_plan` 参数，跳过第一步，直接从第二步开始执行。

### 第一步：将验证方案写入 poc_plan.md，等待确认

根据 hypothesis 生成验证方案，写入 `specs/{module}/poc/{slug}/poc_plan.md`：

```markdown
# POC 验证方案

**假设**：{hypothesis}

## 验证维度

列出要测量/观察的指标，例如：响应时间、内存占用、错误率

## 验证方式

描述将写什么最小复现代码（不超过 3 个文件），以及运行方式

## 成功标准

明确的数值或行为，例如：P99 < 100ms

## 失败标准

明确的数值或行为

## 环境依赖

若需要特定环境（数据库、中间件等），说明将使用 Dockerfile / docker-compose.yml
```

文件写入后，在终端提示用户：

```
POC 验证方案已写入 specs/{module}/poc/{slug}/poc_plan.md，请确认后继续。
```

**等待用户确认后再执行第二步。**

### 第二步：执行验证

1. 在 `specs/{module}/poc/{slug}/` 下写最小可运行代码
2. 若验证依赖特定环境（数据库、中间件、运行时版本等），用 `Dockerfile` 或 `docker-compose.yml` 描述环境，通过容器运行验证
3. 运行代码，收集结果
4. 若运行失败，分析原因后调整一次，仍失败则停止并报告

### 第三步：输出结论

将结论写入 `specs/{module}/poc/{slug}/result.md`，并在终端输出摘要：

```
## POC 结论

**假设**：{hypothesis}
**结论**：可行 / 不可行 / 有条件可行

**关键数据**：
- [指标1]：[实测值] vs [成功标准]
- [指标2]：[实测值] vs [成功标准]

**结论依据**：[1-3 句话说明为什么得出此结论]

**如果可行，plan 中应注意**：[影响实现的约束或前提条件，没有则省略]
**如果不可行，建议替代方案**：[1-2 个备选，没有则省略]
```

## 检查清单

- [ ] 验证维度与 hypothesis 直接相关，无多余测量
- [ ] 成功/失败标准在执行前已明确（不是事后定义）
- [ ] POC 代码是最小实现，无生产级错误处理或抽象
- [ ] 结论基于实测数据，不是推断

## 常见错误

- 验证范围蔓延（顺手验证了其他假设）
- 成功标准模糊（"性能还不错"而非具体数值）
- POC 代码过度工程化，偏离验证目的
- 结论含糊，无法直接用于 plan 决策

## 与 generate_plan 的衔接

`generate_plan` 第一步确认摘要中若列出需要 POC 验证的决策点，先运行 `run_poc`，将 `result.md` 路径作为 plan 技术选型的依据引用，再继续 `generate_plan` 第二步。
