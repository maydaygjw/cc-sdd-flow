---
description: 从零初始化 Python SDD 项目结构，生成 CLAUDE.md 框架、venv 环境和目录骨架
argument-hint: "<project_name>" "<python_version>"
---

# /init_project

## 职责边界

为空的 Python 项目建立 SDD 工作流所需的基础结构。只创建骨架，不生成任何业务内容。业务内容由后续的 `/generate_spec` 开始。

## 参数说明

- `$1` — PROJECT_NAME：项目名称，如 `audio-recorder`
- `$2` — PYTHON_VERSION：Python 版本，如 `3.11`

两个参数都可以省略，省略时通过交互式提问获取。

## 执行流程

### 第一步：收集信息

如果参数不完整，依次询问：

```
1. 项目名称是什么？
2. 使用哪个 Python 版本？（如 3.11、3.12，留空默认 3.12）
```

收集完毕后输出确认摘要，等待用户确认后继续：

```
即将初始化 Python 项目：
- 名称：[项目名称]
- Python 版本：[版本号，默认 3.12]

将创建以下内容：
- venv/（Python 虚拟环境）
- requirements.txt / requirements-dev.txt
- .gitignore
- CLAUDE.md
- constitution.md（如不存在）
- specs/ 目录
- .claude/rules/testing.md（如不存在）

确认继续？
```

### 第二步：创建 venv 和依赖文件

```bash
python{version} -m venv venv
```

创建 `requirements.txt`（运行时依赖，初始为空，加注释说明用途）：
```
# 运行时依赖
# 在此添加项目运行所需的包，如：
# requests==2.31.0
```

创建 `requirements-dev.txt`（开发依赖）：
```
# 开发依赖
pytest
pytest-asyncio
```

在 `CLAUDE.md` 的环境管理章节中写明：**所有命令必须在 venv 激活状态下执行**。

### 第三步：创建目录结构和 .gitignore

创建目录：

```
specs/                    # 所有功能规格
src/                      # 源代码（空）
tests/
  unit/                   # 单元测试
  integration/            # 集成测试
  e2e/                    # 端到端测试
.claude/
  rules/
    testing.md            # 测试规范（如不存在则从模板创建）
```

创建 `.gitignore`：

```gitignore
# Python
venv/
__pycache__/
*.pyc
*.pyo
*.pyd
*.egg-info/
dist/
build/
.eggs/

# 测试 & 覆盖率
.pytest_cache/
.coverage
htmlcov/

# 环境变量
.env
.env.*

# 系统
.DS_Store
Thumbs.db
```

`.gitignore` 已存在时跳过，不覆盖。

### 第四步：生成 CLAUDE.md

```markdown
# CLAUDE.md

> **最高优先级**：核心开发原则在根目录 `constitution.md` 中定义。
> 当本文件与 constitution.md 冲突时，以 constitution.md 为准。

## 项目概述

[PROJECT_NAME]

**开发方法论**：Specification Driven Development (SDD) + Test-Driven Development (TDD)

## 技术栈

- 语言：Python [PYTHON_VERSION，默认 3.12]+
- 测试框架：pytest
- 虚拟环境：venv

## 环境管理

**所有命令必须在 venv 激活状态下执行：**

\`\`\`bash
# 激活 venv
source venv/bin/activate      # macOS / Linux
venv\Scripts\activate         # Windows

# 安装依赖
pip install -r requirements.txt
pip install -r requirements-dev.txt

# 运行测试
pytest tests/unit/
pytest tests/integration/
pytest tests/e2e/
\`\`\`

**安装新依赖时**，同步更新 requirements.txt：
\`\`\`bash
pip install {package}
pip freeze | grep {package} >> requirements.txt
\`\`\`

## 项目架构

> 此节在完成首个功能的 /generate_plan 后补充，当前留空。

## 任务执行指引

实现新功能时：

1. **先读 specs**：查看 `./specs/{feature}/spec.md`、`plan.md`、`tasks.md`
2. **按顺序执行**：严格按 `tasks.md` 中的顺序执行
3. **测试先行**：先写失败的测试，再写实现代码
4. **标记进度**：完成任务后更新 `tasks.md` 中的复选框 `[ ]` → `[x]`

## SDD 工作流

新功能开发顺序：

\`\`\`
/generate_spec {module} "{需求描述}"   # 生成业务规格
/generate_plan {module}               # 生成技术方案
/generate_task {module}               # 生成任务列表
\`\`\`

## 错误处理策略

- **底层模块**：抛出具体异常，附带完整上下文
- **中间层**：让异常向上传播，不做无意义的捕获
- **顶层入口**：捕获所有异常，显示友好错误信息，确保资源清理
```

### 第五步：检查 constitution.md

如果根目录没有 `constitution.md`，从插件内置模板创建标准版本并提示：

```
⚠️  未找到 constitution.md，已从模板创建。
    请在开始开发前审阅并根据项目需求调整。
```

已存在则跳过。

### 第六步：输出下一步指引

```
✅ Python 项目初始化完成

创建的文件：
  venv/
  requirements.txt
  requirements-dev.txt
  src/
  tests/unit/ integration/ e2e/
  specs/
  .gitignore
  CLAUDE.md
  .claude/rules/testing.md
  constitution.md（如为新建）

激活虚拟环境：
  source venv/bin/activate      # macOS / Linux
  venv\Scripts\activate         # Windows

下一步：
  描述你的第一个功能需求，然后运行：
  /generate_spec 001-{feature-name} "{需求描述}"

提示：
  - CLAUDE.md 的"项目架构"节在第一个 plan.md 完成后手动补充
  - 每完成一个功能模块，可用 /init 让 Claude Code 更新 CLAUDE.md
```

## 注意事项

- 不覆盖已存在的 `CLAUDE.md` 和 `constitution.md`，存在时提示用户选择：覆盖 / 跳过
- 不生成任何业务代码或测试文件
- `specs/` 只创建空目录，不预创建功能子目录
- venv 已存在时跳过创建，不覆盖
