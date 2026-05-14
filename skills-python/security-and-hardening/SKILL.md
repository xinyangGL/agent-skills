---
name: security-and-hardening
description: 实施安全最佳实践并强化应用程序。当处理用户输入、认证、数据存储或外部集成时使用。当进行安全审查或加固应用程序时使用。当用户要求"使这个更安全"或"审查安全"时使用。
---

# 安全与加固

## 概览

实施安全最佳实践并强化应用程序。安全不是一次性检查——它是每一层架构中的持续纪律。 OWASP Top 10 是最低基线，不是目标。

## 何时使用

- 处理用户输入时
- 实现认证或授权时
- 存储或处理敏感数据时
- 集成外部服务或 API 时
- 进行安全审查时
- 用户要求加固应用程序时

## 三层边界系统

```
第 3 层（最外层）：  用户输入、请求、URL、查询参数
                    ↓ 验证 + 清理
第 2 层（中间层）：  已验证的数据、服务调用、数据库查询
                    ↓ 信任，但仍进行边界检查
第 1 层（最内层）：  核心业务逻辑、内部工具函数
                    ↓ 信任，无重复验证
```

**规则：** 只在边界处验证。内部代码信任已验证的数据。

## OWASP Top 10 预防

### 1. 注入（SQL、NoSQL、OS 命令、LDAP）

```python
# 好：参数化查询（SQLAlchemy）
def find_by_email(email: str) -> Task:
    return session.query(Task).filter(Task.email == email).first()  # 参数化，安全

# 避免：字符串拼接（常见错误）
def find_by_email_unsafe(email: str) -> Task:
    return session.execute(  # SQL 注入风险
        f"SELECT * FROM tasks WHERE email = '{email}'"
    ).fetchone()
```

**规则：** 永远不要将用户输入拼接到查询、命令或表达式中。

### 2. 认证失效

```python
# 好：强密码哈希（passlib + bcrypt）
from passlib.context import CryptContext

pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")

def hash_password(password: str) -> str:
    return pwd_context.hash(password)  # 安全哈希

def verify_password(password: str, hashed: str) -> bool:
    return pwd_context.verify(password, hashed)

# 好：会话安全（FastAPI）
from fastapi.security import OAuth2PasswordBearer

oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")
# 或使用 sessions
session_cookie_config = {
    "httponly": True,    # JavaScript 无法访问
    "secure": True,      # 仅 HTTPS
    "samesite": "lax",   # CSRF 保护
}
```

### 3. 敏感数据暴露

```python
# 好：从 API 响应中排除敏感字段
from pydantic import BaseModel

class SafeUser(BaseModel):
    id: str
    email: str
    name: str

    @classmethod
    def from_user(cls, user: User) -> "SafeUser":
        return cls(id=user.id, email=user.email, name=user.name)

# 好：环境变量中的密钥
from pydantic_settings import BaseSettings

class Settings(BaseSettings):
    api_key: str  # 从环境变量加载
    jwt_secret: str

    class Config:
        env_file = ".env"

# 避免：硬编码密钥
API_KEY = "sk-1234567890abcdef"  # 永远不要这样做
```

### 4. XSS（跨站脚本）

```python
# 好：Jinja2 默认转义
@app.get("/tasks")
async def tasks(user_input: str):
    return templates.TemplateResponse("tasks.html", {"task_title": user_input})
    # Jinja2 默认转义，安全

# 避免：禁用转义
# {{ userInput | safe }}  # 不转义，XSS 风险

# 如果必须使用 HTML，先清理
import bleach
safe_html = bleach.clean(user_input)
```

### 5. 打破的访问控制

```python
# 好：在每个端点上检查授权
@app.get("/api/tasks/{task_id}")
async def get_task(task_id: str, current_user: User = Depends(get_current_user)):
    task = task_service.find_by_id(task_id)
    if task.owner_id != current_user.id:
        raise HTTPException(status_code=403, detail="无权访问")
    return task
```

**规则：** 永远不要仅依赖前端隐藏来保护敏感操作。

### 6. 安全配置错误

```python
# 好：生产环境安全头部
from fastapi.middleware.trustedhost import TrustedHostMiddleware
from starlette.middleware.base import BaseHTTPMiddleware

app.add_middleware(
    BaseHTTPMiddleware,
    lambda request, call_next: SecurityHeadersMiddleware(call_next),
)

class SecurityHeadersMiddleware:
    async def __call__(self, request: Request, call_next):
        response = await call_next(request)
        response.headers["Content-Security-Policy"] = "default-src 'self'; script-src 'self'"
        response.headers["Strict-Transport-Security"] = "max-age=31536000; includeSubDomains"
        response.headers["X-Frame-Options"] = "DENY"
        return response
```

### 7. 安全日志和监控

```python
# 好：安全的错误处理
from fastapi import Request, HTTPException
from fastapi.responses import JSONResponse
import logging

logger = logging.getLogger(__name__)

@app.exception_handler(Exception)
async def handle_exception(request: Request, exc: Exception):
    # 记录详细错误（内部）
    logger.error("内部错误: %s", exc, exc_info=True)

    # 向用户返回通用消息
    return JSONResponse(
        status_code=500,
        content={"error": {"code": "INTERNAL_ERROR", "message": "出了点问题", "details": None}},
    )
```

**规则：** 永远不要向用户暴露堆栈跟踪或内部详细信息。

## 密钥管理

```python
# 好：环境变量（pydantic-settings）
from pydantic_settings import BaseSettings

class Settings(BaseSettings):
    database_url: str
    api_key: str
    jwt_secret: str

    class Config:
        env_file = ".env"
        env_file_encoding = "utf-8"

settings = Settings()

# 好：验证必需的密钥
if not settings.jwt_secret:
    raise ValueError("缺少必需的密钥: JWT_SECRET")
```

**.env.example（提交到版本控制）：**
```
DATABASE_URL=postgresql://user:password@localhost:5432/dbname
API_KEY=your-api-key-here
JWT_SECRET=your-jwt-secret-here
```

## 依赖审计

```bash
# 检查已知漏洞
pip install safety
safety check

# 或
pip-audit

# 自动修复（可能时）
pip install --upgrade <package>

# 检查严重漏洞
safety check --json --output safety-report.json
```

## 速率限制

```python
# 认证端点的速率限制
from slowapi import Limiter
from slowapi.util import get_remote_address

limiter = Limiter(key_func=get_remote_address)

@app.post("/api/auth/login")
@limiter.limit("5 per 15 minutes")
async def login(request: Request):
    # 认证逻辑
    pass
```

## 常见合理化

| 合理化 | 现实 |
|---|---|
| "我们太小了，不是攻击目标" | 自动化攻击针对所有人，不分大小。安全是必需的。 |
| "我们稍后会添加安全" | 安全债务最难偿还。从第一天就开始安全。 |
| "框架处理了安全" | 框架防止了一些问题，但无法修复错误的配置或不安全的代码。 |
| "过度工程可以稍后进行" | 安全不是功能——它是基础。没有它，其他的一切都岌岌可危。 |

## 危险信号

- 硬编码的密钥或密码
- 向用户暴露的堆栈跟踪
- 未验证的用户输入
- 缺少速率限制的认证端点
- 没有 HTTPS
- 在日志中记录的敏感数据
- 缺少 CORS 配置
- `pip-audit` 显示未修复的严重漏洞

## 验证

在安全审查完成后：

- [ ] 所有用户输入在边界处验证
- [ ] 密钥存储在环境变量中（不在代码中）
- [ ] 密码使用强算法哈希（bcrypt、argon2）
- [ ] 会话 cookie 正确配置（httponly、secure、samesite）
- [ ] 错误消息不暴露内部详细信息
- [ ] 安全头部已配置（CSP、HSTS 等）
- [ ] 认证端点有速率限制
- [ ] `pip-audit` 没有严重或高危漏洞
- [ ] CORS 限制为特定来源