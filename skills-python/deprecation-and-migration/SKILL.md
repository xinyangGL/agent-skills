---
name: deprecation-and-migration
description: 以最小中断的方式弃用旧系统并迁移用户。当需要移除旧系统、迁移用户或下线功能时使用。当用户要求"如何弃用这个"、"迁移用户"或"下线功能"时使用。
---

# 弃用与迁移

## 概览

将代码视为负债。每个功能都是一种承诺——用户依赖它。当你移除某些东西时，你的工作是迁移用户而不会中断他们的工作流程。

## 何时使用

- 移除旧 API 或功能
- 迁移用户到新系统
- 下线旧功能
- 替换遗留系统
- 用户要求"如何弃用这个"或"迁移用户"

## 强制性 vs 建议性弃用

| 类型 | 何时使用 | 影响 |
|------|---------|------|
| **强制性** | 功能不安全、有 bug 或违反合规 | 用户必须迁移，有明确截止日期 |
| **建议性** | 更好的替代品存在，但旧功能仍然工作 | 用户选择何时迁移 |

**规则：** 建议性弃用不应该中断现有用户。强制性弃用需要清晰的迁移路径和时间表。

## 迁移模式

### 1. 双运行期（并行运行旧和新）

```
旧系统  →  仍然处理流量
新系统  →  并行处理相同流量
            ↓
         比较结果，验证新系统
            ↓
         切换流量到新系统
```

**优点：** 零停机，可以验证新系统
**缺点：** 需要双倍基础设施

### 2. 特性标志渐进式切换

```python
# settings.py 中的特性标志
from pydantic_settings import BaseSettings

class Settings(BaseSettings):
    feature_new_auth_system: bool = False

settings = Settings()

# 阶段 1：旧系统默认
@app.get("/auth")
async def auth(request: AuthRequest):
    if settings.feature_new_auth_system:
        return await new_auth_service.authenticate(request)
    return await legacy_auth_service.authenticate(request)

# 阶段 2：内部用户切换到新系统
# 阶段 3：10% 用户切换到新系统
# 阶段 4：50% 用户切换到新系统
# 阶段 5：100% 用户切换到新系统
# 阶段 6：移除旧系统
```

**优点：** 渐进式风险，基于指标的决策
**缺点：** 需要特性标志基础设施

### 3. 逐步弃用通知

```
第 1 个月：  添加弃用警告（不中断）
第 2 个月：  增加警告频率，提供迁移指南
第 3 个月：  发送迁移截止日期通知
第 4 个月：  旧系统开始返回错误
第 5 个月：  移除旧系统
```

**优点：** 用户有充足时间准备
**缺点：** 需要维护两个系统更长时间

## 弃用通知

提供清晰、具体的弃用通知：

```python
# API 响应中的弃用头
from fastapi import APIRouter, Response
from fastapi.responses import JSONResponse

@app.get("/api/v1/tasks", deprecated=True)
async def get_tasks_v1(response: Response):
    response.headers["Deprecation"] = "true"
    response.headers["Sunset"] = "Wed, 31 Dec 2025 23:59:59 GMT"
    response.headers["Link"] = '<https://docs.example.com/migration>; rel="successor"'
    return await task_service.find_all()

# 响应体中的弃用警告
from pydantic import BaseModel
from typing import List, Optional

class DeprecationWarning(BaseModel):
    code: str
    message: str
    documentation: Optional[str] = None

class TaskResponse(BaseModel):
    data: List[Task]
    warnings: List[DeprecationWarning]
```

## 迁移指南

每个弃用都应该有迁移指南：

```markdown
# 从 v1 任务 API 迁移到 v2

## 变更摘要
- 端点从 `/api/v1/tasks` 更改为 `/api/v2/tasks`
- 响应格式从 `camelCase` 更改为 `snake_case`
- 认证从 API 密钥更改为 JWT

## 迁移步骤

### 1. 更新端点
```diff
- url = "https://api.example.com/api/v1/tasks"
+ url = "https://api.example.com/api/v2/tasks"
```

### 2. 更新响应处理
```diff
- title = response["data"]["taskList"][0]["title"]
+ title = response["data"]["task_list"][0]["task_title"]
```

### 3. 更新认证
```diff
- headers = {"X-API-Key": api_key}
+ headers = {"Authorization": f"Bearer {jwt_token}"}
```

## 常见问题
- [迁移后我的旧 API 密钥还有效吗？](#q1)
- [v2 API 支持分页吗？](#q2)

## 需要帮助？
查看 [完整迁移文档](https://docs.example.com/migration) 或联系支持。
```

## 僵尸代码清理

迁移后，清理旧代码：

```bash
# 识别未使用的代码
git log --all --oneline --source --remotes --grep="deprecated"
# 检查是否有引用
git grep "legacy_auth_service"
# 安全删除
git rm src/app/auth/legacy_auth_service.py
git commit -m "chore: 移除已弃用的认证系统"
```

**规则：** 在迁移完成后至少等待一个发布周期再删除旧代码。这为意外回滚提供了缓冲。

## 回滚计划

每个迁移都应该有回滚计划：

```markdown
# 迁移回滚计划

## 触发条件
- 错误率 > 1%
- 响应时间 p95 > 500ms
- 用户报告关键功能失败

## 回滚步骤
1. 将特性标志 `feature_new_auth_system` 设置为 `False`
2. 验证流量已切换回旧系统
3. 调查新系统中的问题
4. 修复后重新部署

## 回滚验证
- [ ] 错误率回到基线
- [ ] 响应时间在可接受范围内
- [ ] 用户可以正常登录
```

## 常见合理化

| 合理化 | 现实 |
|---|---|
| "我们可以直接切换" | 直接切换中断用户。渐进式迁移更安全。 |
| "没有人使用那个旧功能" | 直到你移除它并发现有人使用。通知所有用户。 |
| "迁移指南太复杂" | 没有迁移指南的弃用是用户的主意差。投资指南。 |
| "我们可以稍后清理" | 僵尸代码累积技术债务。在迁移后清理。 |

## 危险信号

- 没有通知用户的弃用
- 没有迁移指南的 API 变更
- 没有回滚计划的迁移
- 在迁移前不双运行验证
- 移除代码后不等待一个发布周期
- 没有监控迁移影响

## 验证

迁移后：

- [ ] 用户已收到弃用通知
- [ ] 迁移指南已提供
- [ ] 双运行验证已执行
- [ ] 特性标志控制迁移进度
- [ ] 监控已设置以检测问题
- [ ] 回滚计划已测试
- [ ] 旧代码在缓冲期后已清理