---
name: documentation-and-adrs
description: 创建维护良好的文档和架构决策记录。当做出架构决策、更改 API 或发布功能时使用。当用户要求"记录这个决策"、"为什么我们选择了 X"或"更新文档"时使用。
---

# 文档与 ADRs

## 概览

记录架构决策的 *为什么*，而不仅仅是 *是什么*。代码告诉你系统做什么——文档告诉你为什么以那种方式构建。没有文档，未来的维护者会重新发明你已经做出的决策。

## 何时使用

- 做出架构决策
- 更改 API 或数据库模式
- 引入新的依赖或模式
- 发布新功能
- 用户要求"记录这个决策"
- 你注意到自己在解释相同的决策多次

## 架构决策记录（ADRs）

每个重要的架构决策都应该有一个 ADR：

```markdown
# [决策编号]：[决策标题]

**日期：** [决定日期]
**状态：** 已接受 | 弃用 | 超期
**上下文：** [我们面临什么问题？]
**决策：** [我们决定什么？]
**后果：** [这个决策意味着什么？]

## 背景

[问题的详细描述。为什么这个决策是必要的？]

## 选项

### 选项 1：[名称]
**优点：**
- [优点 1]
- [优点 2]

**缺点：**
- [缺点 1]
- [缺点 2]

### 选项 2：[名称]
**优点：**
- [优点 1]
- [优点 2]

**缺点：**
- [缺点 1]
- [缺点 2]

## 决策

选择 [选项 X]，因为 [理由]。

## 后果

- [积极后果 1]
- [积极后果 2]
- [需要管理的消极后果]

## 相关决策

- [ADR-001：相关决策 1](../001-related-decision.md)
- [ADR-002：相关决策 2](../002-related-decision.md)
```

## ADR 编号和位置

```
docs/
  adr/
    001-use-fastapi.md
    002-use-postgresql.md
    003-use-alembic.md
    ...
```

**编号规则：**
- 从 001 开始，每个新决策递增
- 即使 ADR 被删除，也不要重用编号
- 文件名应描述决策，而不只是编号

## 文档层次结构

```
README.md              # 项目概述、快速开始
docs/
  architecture.md      # 高层架构概述
  adr/                 # 架构决策记录
  api.md              # API 文档
  deployment.md       # 部署指南
  contributing.md     # 贡献指南
```

## API 文档

每个公共 API 都应该有文档：

```markdown
# 任务 API

## 端点

### 创建任务

**POST** `/api/v2/tasks`

**请求体：**
```json
{
  "title": "string (required)",
  "description": "string (optional)",
  "priority": "enum: low | medium | high (optional, default: medium)"
}
```

**响应：**
```json
{
  "id": "string",
  "title": "string",
  "description": "string | null",
  "priority": "string",
  "status": "string",
  "created_at": "ISO 8601 datetime",
  "updated_at": "ISO 8601 datetime"
}
```

**错误：**
- `422` - 验证失败（title 为空或超过 200 字符）
- `401` - 未认证
- `500` - 服务器错误

**示例：**
```bash
curl -X POST https://api.example.com/api/v2/tasks \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"title": "完成任务"}'
```
```

## Python Docstring 标准

```python
def create_task(input: CreateTaskInput) -> Task:
    """
    创建新任务并返回带有服务器生成字段的已创建任务。

    Args:
        input: 任务创建数据

    Returns:
        带有服务器生成 ID 和日期的已创建任务

    Raises:
        ValidationException: 如果 title 为空或超过 200 字符
        DatabaseException: 如果数据库插入失败

    Example:
        >>> task = task_service.create_task(CreateTaskInput(
        ...     title="完成任务",
        ...     description="完成项目文档",
        ...     priority="high"
        ... ))
        >>> task.id
        'task_abc123'
    """
    # 实现
    pass
```

**规则：**
- 每个公共函数都有 docstring 注释
- 描述 *为什么* 和 *什么*，不描述 *如何*（代码已经展示了 *如何*）
- 包括示例对于复杂或容易出错的 API
- 记录抛出的异常和错误条件
- 使用 Google 风格或 Sphinx 风格 docstring（选择一种并保持一致）

## 文档维护

文档会过时。防止这个：

```yaml
# CI 中的文档检查
jobs:
  docs:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: 检查文档链接
        run: npx markdown-link-check docs/**/*.md
      - name: 检查代码示例
        run: pytest tests/test_doc_examples.py
```

**规则：**
- 每次 PR 中，如果代码变更影响文档，更新文档
- 在 CI 中检查文档链接
- 测试代码示例确保它们仍然有效
- 在代码变更时审查相关 ADRs

## 常见合理化

| 合理化 | 现实 |
|---|---|
| "代码是自文档的" | 代码告诉你 *什么*，不告诉你 *为什么*。文档记录推理。 |
| "我们可以稍后写文档" | 稍后意味着从未。在决策时写文档——推理还新鲜。 |
| "文档太繁琐" | 简短的 ADR（5 分钟）节省未来的维护者数小时的困惑。 |
| "这个决策很明显" | 对你来说很明显，对三个月后的你或新团队成员来说可能不是。 |

## 危险信号

- 没有记录的架构决策
- 过时的文档（不匹配代码）
- 断开的文档链接
- 不工作的代码示例
- 没有 ADR 的重大变更
- 只记录 *什么*，不记录 *为什么*

## 验证

文档后：

- [ ] ADR 记录了决策的上下文、选项和后果
- [ ] API 端点有请求/响应示例
- [ ] 公共函数有 docstring 注释
- [ ] 文档链接有效
- [ ] 代码示例经过测试
- [ ] 相关 ADRs 被交叉引用
- [ ] 文档与代码变更一起提交