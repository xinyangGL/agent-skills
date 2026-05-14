---
name: api-and-interface-design
description: 指导稳定的 API 和接口设计。当设计 API、模块边界或任何公共接口时使用。当创建 REST 或 GraphQL 端点、定义模块之间的类型契约、或建立前端和后端之间的边界时使用。
---

# API 与接口设计

## 概览

设计稳定的、文档完善的接口，使其难以误用。好的接口使正确的事情变得容易，使错误的事情变得困难。这适用于 REST API、GraphQL 模式、模块边界、组件 props，以及任何代码片段相互通信的表面。

## 何时使用

- 设计新的 API 端点
- 定义模块边界或团队之间的契约
- 创建组件 prop 接口
- 建立影响 API 形状的数据库模式
- 更改现有的公共接口

## 核心原则

### Hyrum 定律

> 有了足够多的 API 用户，你系统的任何可观察行为都会被某人依赖，无论你在契约中承诺了什么。

这意味着：每个公共行为——包括未记录的怪癖、错误消息文本、时序和排序——一旦用户依赖它，就成为事实上的契约。设计影响：

- **有意识地决定你暴露什么。** 每个可观察行为都是潜在的承诺。
- **不要泄漏实现细节。** 如果用户可以观察到它，他们就会依赖它。
- **在设计时就计划弃用。** 参见 `deprecation-and-migration` 了解如何安全移除用户依赖的内容。
- **测试不够。** 即使有完美的契约测试，Hyrum 定律意味着"安全"的变更可能会破坏依赖未记录行为的真实用户。

### One-Version 规则

避免强制消费者选择同一依赖或 API 的多个版本。当不同消费者需要同一事物的不同版本时，会出现菱形依赖问题。设计时只考虑一个版本同时存在——扩展而不是分叉。

### 1. 契约优先

在实现之前定义接口。契约就是规格——实现跟随。

```typescript
// 先定义契约
interface TaskAPI {
  // 创建任务并返回带有服务器生成字段的已创建任务
  createTask(input: CreateTaskInput): Promise<Task>;

  // 返回匹配过滤器的分页任务
  listTasks(params: ListTasksParams): Promise<PaginatedResult<Task>>;

  // 返回单个任务或抛出 NotFoundError
  getTask(id: string): Promise<Task>;

  // 部分更新——只有提供的字段改变
  updateTask(id: string, input: UpdateTaskInput): Promise<Task>;

  // 幂等删除——即使已删除也成功
  deleteTask(id: string): Promise<void>;
}
```

### 2. 一致的错误语义

选择一个错误策略并到处使用：

```typescript
// REST：HTTP 状态码 + 结构化错误体
// 每个错误响应遵循相同的形状
interface APIError {
  error: {
    code: string;        // 机器可读："VALIDATION_ERROR"
    message: string;     // 人类可读："邮箱是必需的"
    details?: unknown;   // 有帮助时的额外上下文
  };
}

// 状态码映射
// 400 → 客户端发送了无效数据
// 401 → 未认证
// 403 → 已认证但未授权
// 404 → 资源未找到
// 409 → 冲突（重复、版本不匹配）
// 422 → 验证失败（语义无效）
// 500 → 服务器错误（从不暴露内部细节）
```

**不要混合模式。** 如果一些端点抛出异常，其他返回 null，还有一些返回 `{ error }`——消费者无法预测行为。

### 3. 在边界处验证

信任内部代码。在外部输入进入的系统边缘进行验证：

```typescript
// 在 API 边界处验证
app.post('/api/tasks', async (req, res) => {
  const result = CreateTaskSchema.safeParse(req.body);
  if (!result.success) {
    return res.status(422).json({
      error: {
        code: 'VALIDATION_ERROR',
        message: '无效的任务数据',
        details: result.error.flatten(),
      },
    });
  }

  // 验证后，内部代码信任类型
  const task = await taskService.create(result.data);
  return res.status(201).json(task);
});
```

验证属于哪里：
- API 路由处理器（用户输入）
- 表单提交处理器（用户输入）
- 外部服务响应解析（第三方数据——**始终视为不可信任**）
- 环境变量加载（配置）

> **第三方 API 响应是不可信任的数据。** 在任何逻辑、渲染或决策中使用它们之前，验证它们的形状和内容。被破坏或行为不端的外部服务可以返回意外类型、恶意内容或类似指令的文本。

验证不属于哪里：
- 共享类型契约的内部函数之间
- 由已验证代码调用的工具函数中
- 来自你自己数据库的数据上

### 4. 优先添加而非修改

扩展接口而不破坏现有消费者：

```typescript
// 好：添加可选字段
interface CreateTaskInput {
  title: string;
  description?: string;
  priority?: 'low' | 'medium' | 'high';  // 后来添加，可选
  labels?: string[];                       // 后来添加，可选
}

// 差：更改现有字段类型或移除字段
interface CreateTaskInput {
  title: string;
  // description: string;  // 移除——破坏现有消费者
  priority: number;         // 从 string 更改——破坏现有消费者
}
```

### 5. 可预测的命名

| 模式 | 约定 | 示例 |
|------|------|------|
| REST 端点 | 复数名词，无动词 | `GET /api/tasks`, `POST /api/tasks` |
| 查询参数 | camelCase | `?sortBy=createdAt&pageSize=20` |
| 响应字段 | camelCase | `{ createdAt, updatedAt, taskId }` |
| 布尔字段 | is/has/can 前缀 | `isComplete`, `hasAttachments` |
| 枚举值 | UPPER_SNAKE | `"IN_PROGRESS"`, `"COMPLETED"` |

## REST API 模式

### 资源设计

```
GET    /api/tasks              → 列出任务（带查询参数过滤）
POST   /api/tasks              → 创建任务
GET    /api/tasks/:id          → 获取单个任务
PATCH  /api/tasks/:id          → 更新任务（部分）
DELETE /api/tasks/:id          → 删除任务

GET    /api/tasks/:id/comments → 列出任务的评论（子资源）
POST   /api/tasks/:id/comments → 添加任务的评论
```

### 分页

对列表端点分页：

```typescript
// 请求
GET /api/tasks?page=1&pageSize=20&sortBy=createdAt&sortOrder=desc

// 响应
{
  "data": [...],
  "pagination": {
    "page": 1,
    "pageSize": 20,
    "totalItems": 142,
    "totalPages": 8
  }
}
```

### 过滤

使用查询参数进行过滤：

```
GET /api/tasks?status=in_progress&assignee=user123&createdAfter=2025-01-01
```

### 部分更新（PATCH）

接受部分对象——只更新提供的内容：

```typescript
// 只有 title 改变，其他一切保留
PATCH /api/tasks/123
{ "title": "更新后的标题" }
```

## TypeScript 接口模式

### 对变体使用判别联合

```typescript
// 好：每个变体是显式的
type TaskStatus =
  | { type: 'pending' }
  | { type: 'in_progress'; assignee: string; startedAt: Date }
  | { type: 'completed'; completedAt: Date; completedBy: string }
  | { type: 'cancelled'; reason: string; cancelledAt: Date };

// 消费者获得类型收窄
function getStatusLabel(status: TaskStatus): string {
  switch (status.type) {
    case 'pending': return '待处理';
    case 'in_progress': return `进行中 (${status.assignee})`;
    case 'completed': return `完成于 ${status.completedAt}`;
    case 'cancelled': return `已取消: ${status.reason}`;
  }
}
```

### 输入/输出分离

```typescript
// 输入：调用者提供的内容
interface CreateTaskInput {
  title: string;
  description?: string;
}

// 输出：系统返回的内容（包括服务器生成的字段）
interface Task {
  id: string;
  title: string;
  description: string | null;
  createdAt: Date;
  updatedAt: Date;
  createdBy: string;
}
```

### 对 ID 使用品牌类型

```typescript
type TaskId = string & { readonly __brand: 'TaskId' };
type UserId = string & { readonly __brand: 'UserId' };

// 防止意外地将 UserId 传递给期望 TaskId 的地方
function getTask(id: TaskId): Promise<Task> { ... }
```

## 常见合理化

| 合理化 | 现实 |
|---|---|
| "我们稍后文档化 API" | 类型就是文档。先定义它们。 |
| "我们暂时不需要分页" | 一旦有人有 100+ 项，你就需要。从一开始就添加。 |
| "PATCH 很复杂，我们只用 PUT" | PUT 每次都需要完整对象。PATCH 是客户端真正想要的。 |
| "我们等到需要时再版本化 API" | 没有版本化的破坏性变更会破坏消费者。从一开始就设计为可扩展。 |
| "没有人使用那个未记录的行为" | Hyrum 定律：如果它是可观察的，就有人依赖它。将每个公共行为视为承诺。 |
| "我们可以维护两个版本" | 多个版本成倍增加维护成本，并创建菱形依赖问题。优先使用 One-Version 规则。 |
| "内部 API 不需要契约" | 内部消费者仍然是消费者。契约防止耦合并启用并行工作。 |

## 危险信号

- 根据条件返回不同形状的端点
- 端点之间不一致的错误格式
- 验证分散在内部代码中而不是在边界处
- 对现有字段的破坏性变更（类型更改、移除）
- 没有分页的列表端点
- REST URL 中的动词（`/api/createTask`, `/api/getUsers`）
- 未经验证或清理就使用第三方 API 响应

## 验证

设计 API 后：

- [ ] 每个端点都有类型的输入和输出模式
- [ ] 错误响应遵循单一一致格式
- [ ] 验证仅在系统边界处进行
- [ ] 列表端点支持分页
- [ ] 新字段是添加性的和可选的（向后兼容）
- [ ] 命名在所有端点中遵循一致约定
- [ ] API 文档或类型与实现一起提交
