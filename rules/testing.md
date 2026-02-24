# 测试规范

---

## 测试分层

| 层次 | 测试对象 | Mock 策略 | 数量约束 |
|------|---------|----------|---------|
| 单元测试 | 纯函数（无副作用、无 I/O） | 禁止任何 Mock | 每个方法最多 3 个用例 |
| 集成测试 | I/O 操作、模块协作、外部系统交互 | 可 Mock 外部依赖，禁止 Mock 内部逻辑 | 每个任务最多 1 类 3 个方法 |
| E2E 测试 | 完整业务流程 | 禁止任何 Mock，使用真实或容器化环境 | 仅覆盖主流程 |

---

## 纯函数判定

以下条件全部满足才是纯函数，才可以写单元测试：

- 相同输入总是产生相同输出
- 无副作用（不修改外部状态）
- 不依赖外部状态、不进行 I/O

有副作用的方法（数据库、API、文件、浏览器交互）在集成测试中测试，不写单元测试。

**async 函数**：`async` 关键字本身不影响判定。无 I/O 的纯 async 函数可写单元测试；涉及 I/O 的放集成测试；混合型应重构，将纯逻辑提取为同步纯函数后单元测试。

---

## 命名规范

**测试类**：`Test{TargetClass}`，每个被测类对应一个测试类

**测试方法**：`test_{method_name}_{scene}`，场景建议：
- `_valid_input` / `_happy_path`（主流程）
- `_edge_cases` / `_boundary`（边界条件）
- `_invalid_input` / `_raises_error`（异常场景）

---

## 参数化测试

相同逻辑的多种场景用参数化测试表达，不重复代码：

```python
@pytest.mark.parametrize("time_str,expected", [
    ("00:30", 30),
    ("01:00", 60),
    ("59:59", 3599),
])
def test_parse_time_string_valid_input(time_str, expected):
    assert parse_time_string(time_str) == expected
```

---

## TDD 循环

Red → Green → Refactor，不可跳步。细节见 `constitution.md` 第二条。