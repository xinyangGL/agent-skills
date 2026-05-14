---
name: test-driven-development
description: 通过测试驱动实现逻辑和修复 bug。当逻辑需要实现或 bug 需要修复时使用。当用户要求编写测试时使用。当行为需要证据而非信任时使用。
---

# 测试驱动开发

## 概览

通过编写先失败的测试来推动代码前进——然后编写足够的代码使其通过，再重复。测试即证明。如果它没有测试，它不存在。

## 何时使用

- 需要实现新逻辑时
- 需要修复 bug 时
- 用户要求为某些内容编写测试时
- 你需要证明行为，而非依赖信任时

## 何时不使用

- 当用户明确要求快速/草稿原型且接受无测试的权衡时（记录此决定）
- 纯 UI 布局或样式工作，没有行为逻辑时（但仍应测试行为，如键盘导航）

## 测试金字塔

```
        /    \         ← E2E：少量，覆盖关键用户流程
       /------\        ← 集成：中等，覆盖跨越边界的交互
      /--------\       ← 单元：大量，覆盖纯逻辑
     /----------\
```

- **单元（80%）：** 测试纯逻辑，没有 I/O。快速，隔离，描述性。
- **集成（15%）：** 测试跨越边界的交互——数据库、网络、服务。
- **E2E（5%）：** 测试关键用户流程，从用户的视角。

## 测试大小：测试即规格

保持测试小巧，使其成为可读的规格，而不是需要调试的实现。

```python
# 好：一个断言，一个概念
def test_rejects_tasks_with_empty_titles():
    result = task_service.create_task(TaskRequest(title="", description=None))
    assert result.error.code == "INVALID_TITLE"

# 避免：多个概念，像在生产中一样编排
def test_handles_full_creation_flow():
    user = user_service.create_user(...)
    auth = auth_service.authenticate(user)
    task = task_service.create_task(TaskRequest(title="Test"), auth)
    assert task.id is not None
    assert task.title == "Test"
    assert task.status == "PENDING"
    updated = task_service.update_task(task.id, TaskUpdate(status="done"), auth)
    assert updated.status == "DONE"
```

小测试更容易阅读、更快运行、更不容易因无关原因失败。

## 处理现有代码

当为现有代码编写测试时，从冒险测试开始——风险最高的代码、最可能引入 bug 的代码。

```
高风险，无测试  → 优先（数据处理、认证、状态转换）
低风险，无测试  → 稍后（工具函数、格式化、简单渲染）
已有测试        → 跳过，除非测试不充分
```

对于冒险测试，遵循 Prove-It 模式：
1. 编写一个使用现有代码的测试
2. 确认它通过（当前行为）
3. 现在你可以安全地重构，测试会捕获任何回归

对于 bug 修复，使用"演示 bug"测试：
1. 编写一个使用当前代码的测试
2. 测试**必须**失败——如果它通过，它没有测试 bug
3. 修复代码，确认测试通过

## 测试命名

```python
class TestCreateTask:
    """测试任务创建行为"""

    def test_rejects_tasks_with_empty_titles(self):
        # Arrange → Act → Assert
        ...

    def test_creates_task_with_default_pending_status(self):
        ...
```

每个测试名称应读起来像一份规格——描述正在验证的行为，而不是实现细节。

## 测试顺序

```
1. 单元     → 纯逻辑，无 I/O，无框架
2. 集成     → 跨越边界：数据库、网络、服务
3. E2E      → 关键用户流程，浏览器或真实 HTTP
```

在能捕获行为的最低层级进行测试。不要用 E2E 测试去覆盖单元测试可以覆盖的内容。

## Mock 的位置

在系统边界处（数据库、网络、外部服务）进行 Mock，而不是在内部函数之间。

```python
# 好：在数据库边界处 Mock
def test_returns_tasks_from_database(mocker):
    mock_repo = mocker.patch("app.repository.TaskRepository.find_all")
    mock_repo.return_value = [Task(id=1, title="Test")]
    result = task_service.get_tasks()
    assert len(result) == 1

# 避免：Mock 内部逻辑
def test_calls_the_repository(mocker):
    # 现在测试依赖于内部实现
    pass
```

## 测试依赖

- 测试**不要**依赖外部服务（使用 Mock 或本地测试数据库）
- 测试**不要**依赖执行顺序（每个测试是独立的）
- 测试**不要**依赖特定时间（使用可注入的时钟）
- 测试**不要**依赖文件系统（使用内存替代）

## DAMP 优于 DRY

测试应该是 DAMP（浅而易读），而不是 DRY（不要重复自己）。

```python
# 好：DAMP，每个测试独立且可读
def test_marks_task_as_done():
    tasks = task_tracker.add(TaskRequest(title="Test"))
    assert tasks[0].status == "PENDING"
    task_tracker.complete(tasks[0].id)
    assert tasks[0].status == "DONE"

def test_cannot_complete_task_twice():
    tasks = task_tracker.add(TaskRequest(title="Test"))
    task_tracker.complete(tasks[0].id)
    task_tracker.complete(tasks[0].id)
    assert tasks[0].status == "DONE"
```

## Beyonce 规则

测试应该只测试它们自己，不测试其他东西。如果一个测试失败，你应该能够修复它而不修改其他测试。

```
✓ 每个测试独立
✓ 每个测试覆盖一个概念
✗ 测试之间共享可变状态
✗ 一个测试的结果影响另一个
```

## 浏览器测试

使用 pytest-playwright 进行真实的运行时浏览器测试——验证应用程序在浏览器中实际运行。

```python
# 浏览器测试示例——使用 pytest-playwright 运行
def test_task_creation_flow_works_in_browser(page: Page):
    page.goto("http://localhost:3000/tasks")
    page.click('[data-testid="new-task-button"]')
    page.fill('[data-testid="task-title"]', "Test task")
    page.click('[data-testid="submit-button"]')
    page.wait_for_selector('[data-testid="task-list"]')
    assert "Test task" in page.text_content('[data-testid="task-list"]')
```

## 常见合理化

| 合理化 | 现实 |
|---|---|
| "我会稍后添加测试" | 稍后意味着从未。在编写代码之前编写测试。 |
| "这太简单了，不需要测试" | 简单的东西会变化。测试防止简单变成脆弱。 |
| "测试减慢了我的速度" | 没有测试，你不知道你是否弄坏了东西。测试是速度，不是减速。 |
| "我手动测试了，它工作了" | 手动测试不会运行于 CI 中。手动测试不可重复。手动测试不是测试。 |
| "这个函数是私有的，所以我不测试它" | 私有逻辑仍然需要验证。通过公共接口测试它，或直接测试它。 |
| "我会用 E2E 覆盖这个" | E2E 缓慢且脆弱。在最低层级测试。E2E 仅用于关键用户流程。 |

## 危险信号

- 在编写实现代码之前没有失败的测试
- 测试依赖执行顺序或共享可变状态
- 所有测试都是 E2E（没有单元或集成测试）
- 测试断言内部状态而非可观察行为
- 跳过测试，理由是"这很简单"
- 断言"它不崩溃"而没有验证具体行为
- 在生产代码编写完成"之后"编写测试

## 验证

在任何东西完成之前：

- [ ] 测试在实现之前编写并失败
- [ ] 现在所有测试通过
- [ ] 测试覆盖正常路径和边界情况
- [ ] 测试独立运行（不依赖顺序或共享状态）
- [ ] 没有外部依赖未进行 Mock（除非是专门的集成测试）
- [ ] 浏览器测试验证了关键用户流程（如适用）