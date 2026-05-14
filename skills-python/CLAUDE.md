# Python 项目配置模板 (CLAUDE.md)

# 项目：[项目名称]

## 技术栈
- Python 3.12, FastAPI, uvicorn, PostgreSQL, Alembic
- pytest, httpx, pytest-playwright
- Pydantic, SQLAlchemy

## 命令
- 构建：`pip install -e .`
- 测试：`pytest`
- Lint：`ruff check . && black .`
- 开发：`uvicorn app.main:app --reload`
- 类型检查：`mypy src/`
- 依赖审计：`pip-audit`

## 代码约定
- 遵循 PEP 8
- 使用 type hints
- 使用 Pydantic 进行请求/响应验证
- 测试与源码共存：`task_service.py` → `test_task_service.py`
- 使用 FastAPI 依赖注入处理认证和授权
- 使用 SQLAlchemy 进行数据访问
- 使用 logging 模块（logger.debug()）

## 项目结构
```
src/app/
  routes/           # API 路由
    tasks.py
  services/         # 业务逻辑
    task_service.py
  models/           # 数据模型
    task.py
  schemas/          # Pydantic 验证模式
    task.py
  utils/            # 共享工具函数
    validation.py
  config/           # 配置
    settings.py
tests/
  unit/             # 单元测试
    test_task_service.py
  integration/      # 集成测试
    test_task_routes.py
  e2e/              # E2E 测试
    test_task_flow.py
```

## 测试策略
- 单元测试：所有 Service 层方法（80%+ 覆盖）
- 集成测试：所有 API 端点（pytest + httpx）
- E2E 测试：关键用户流程（pytest-playwright）

## 边界
- 绝不提交 .env 文件或密钥
- 绝不添加依赖而不检查包大小影响
- 修改数据库模式前询问
- 提交前始终运行测试
- 始终使用参数化查询（防止 SQL 注入）
- 始终使用 passlib + bcrypt 哈希密码
- 始终在 API 边界处验证输入（Pydantic 模型）

## 模式示例
```python
from fastapi import APIRouter, status
from app.schemas.task import CreateTaskInput, Task

router = APIRouter()

@router.post("/api/tasks", status_code=status.HTTP_201_CREATED)
async def create_task(input: CreateTaskInput):
    task = await task_service.create(input)
    return task
```

## 安全约定
- 密码使用 passlib + bcrypt 哈希
- 会话 cookie 配置：httponly、secure、samesite
- 认证端点速率限制（slowapi）
- 错误消息不暴露内部细节
- CORS 限制为特定来源
- 使用 Starlette Middleware 配置安全头部

## CI/CD
- 每次 PR 自动运行：`pytest && ruff check . && mypy src/`
- 包含：lint、单元测试、集成测试、依赖审计
- 构建失败则阻止合并

## 性能预算
- API 响应时间：< 200ms（p95）
- 数据库查询：< 50ms（p95）
- Python 进程内存：< 512MB（生产环境）
- GIL 争用：< 10%

## Git 工作流
- 主干开发（trunk-based）
- 原子提交（~100 行/提交）
- 功能分支短寿命（< 2 天）
- 使用特性标志进行长期工作
- 提交消息格式：`feat: 添加任务创建端点`