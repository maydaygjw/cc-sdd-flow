---
description: 将需求记录到 specs/backlog.md，作为待处理需求的备忘清单
argument-hint: "<requirement>"
---

# /add_backlog

## 职责边界

将用户描述的需求以结构化条目追加到 `specs/backlog.md`，作为备忘和待办清单。不做需求分析，不生成 spec，只做记录。

## 参数说明

- `$1` — REQUIREMENT：需求描述（自然语言），可省略，省略时交互式输入。

## 执行流程

### 第一步：收集信息

如果未传入参数，询问：

```
请描述这条需求（一句话或几句话均可）：
```

### 第二步：输出确认摘要

提炼需求后输出确认，等待用户确认后继续：

```
即将记录以下 backlog 条目，请确认：
- 标题：[一句话概括]
- 描述：[原始描述或简化后的描述]
- 优先级：[High / Medium / Low，根据描述语气判断，默认 Medium]
```

如用户对优先级或标题有异议，按用户意见修改后再写入。

### 第三步：写入 backlog.md

**若 `specs/backlog.md` 不存在**，先创建并写入文件头：

```markdown
# Backlog

需求备忘清单，按记录时间倒序排列。通过 `/add_backlog` 追加条目，通过 `/generate_spec` 将条目转化为正式规格。

---
```

**追加新条目**（插入到 `---` 分隔线之后、已有条目之前，保持倒序）：

```markdown
## [BL-NNN] <标题>

- 状态：`todo`
- 优先级：`High` / `Medium` / `Low`
- 记录时间：YYYY-MM-DD
- 关联模块：（如有，填写 specs 目录下的模块名；否则留空）

**描述**

<用户原始描述或整理后的描述>

**备注**

（暂无）

---
```

其中 `BL-NNN` 为自增编号，读取文件中已有的最大编号后 +1，从 `BL-001` 开始。

### 第四步：输出结果

```
已记录：[BL-NNN] <标题>
文件：specs/backlog.md
```

## 状态说明

| 状态 | 含义 |
|------|------|
| `todo` | 待处理，尚未开始 |
| `in-progress` | 已创建对应 spec，正在开发 |
| `done` | 已完成并上线 |
| `dropped` | 已放弃，不再处理 |

状态由用户手动维护，或在执行 `/generate_spec` 时由 Agent 更新为 `in-progress`。

## 后续步骤

backlog 条目就绪后：
1. 运行 `/add_backlog` 继续追加其他需求
2. 运行 `/generate_spec <module> "<BL-NNN 的描述>"` 将某条 backlog 转化为正式规格
