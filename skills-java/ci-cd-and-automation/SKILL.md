---
name: ci-cd-and-automation
description: 实施持续集成和持续部署。当设置或修改构建和部署流水线、添加自动化、或配置 CI/CD 时使用。当用户要求"设置 CI"、"自动化测试"或"部署流水线"时使用。
---

# CI/CD 与自动化

## 概览

设置自动化构建、测试和部署流水线。目标是通过左移质量门禁来更快、更安全地发布。每次提交都应该自动验证，每个部署都应该是可重复和可回滚的。

## 何时使用

- 设置新的 CI/CD 流水线
- 修改现有的构建或部署流程
- 添加自动化测试或质量检查
- 配置特性标志
- 设置监控或警报
- 用户要求"设置 CI"或"自动化部署"

## 左移原则

尽早运行检查——在问题变得昂贵修复之前捕获它们。

```
传统：          开发 → 测试 → QA → 部署 → 生产 → 发现问题
左移：          开发 → 自动测试 → 自动部署 → 生产 → 监控
                  ↑ 尽早捕获问题
```

**左移检查：**
- 预提交钩子：lint、格式化、基本测试
- CI 流水线：完整测试套件、类型检查、构建验证
- 预部署：集成测试、性能测试、安全扫描
- 部署后：烟测试、健康检查、监控

## 更快、更安全

自动化不是为了速度——速度是副产物。自动化是为了安全——每次一致地运行相同的检查。

```yaml
# .github/workflows/ci.yml
name: CI

on:
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-java@v4
        with:
          java-version: '17'
          distribution: 'temurin'
      - run: mvn clean test
      - run: mvn spotless:check
      - run: mvn dependency:check -DpluginId=owasp-dependency-check
```

## 特性标志

对于可能影响现有用户的大型变更，使用特性标志：

```java
// Spring Boot 特性标志
@Component
public class FeatureFlags {
    @Value("${feature.new-task-flow:false}")
    private boolean newTaskFlow;

    public boolean isNewTaskFlowEnabled() {
        return newTaskFlow;
    }
}
```

**规则：**
- 特性标志默认关闭，直到准备好
- 在 CI 中测试两种状态（开启和关闭）
- 在全面推出后 2 周内清理

## 质量门禁流水线

```
提交 →  lint → 单元测试 → 构建 → 集成测试 → 部署
         ↓         ↓          ↓         ↓           ↓
       失败      失败       失败      失败        失败
```

每个门禁在下一个开始之前必须通过。如果一个门禁失败，流水线停止——不传播损坏。

## 失败反馈循环

当流水线失败时，反馈应该是快速和具体的：

```yaml
# 失败通知
jobs:
  notify:
    needs: [test]
    if: failure()
    runs-on: ubuntu-latest
    steps:
      - name: 发送失败通知
        run: |
          echo "CI 失败：${{ github.repository }}"
          echo "提交：${{ github.sha }}"
          echo "分支：${{ github.ref }}"
          # 发送到 Slack/Teams/邮件
```

**规则：** 失败应该在 5 分钟内通知团队。延迟的反馈导致更长的修复时间。

## 部署策略

### 蓝绿部署

```
蓝（当前生产）  →  绿（新版本）
                    ↓
               切换流量
                    ↓
蓝（回滚）  ←  绿（新生产）
```

**优点：** 即时回滚，零停机
**缺点：** 需要双倍基础设施

### 金丝雀部署

```
100% 流量 → 版本 A
     ↓
90% A, 10% B  →  监控指标
     ↓
50% A, 50% B  →  监控指标
     ↓
0% A, 100% B  →  完全部署
```

**优点：** 渐进式风险，基于指标的决策
**缺点：** 更复杂的设置

### 滚动部署

```
Pod 1 更新 → Pod 2 更新 → Pod 3 更新 → ...
```

**优点：** 简单，资源高效
**缺点：** 回滚更慢

## 监控设置

```java
// 健康检查端点
@RestController
public class HealthController {

    @GetMapping("/api/health")
    public ResponseEntity<HealthStatus> health() {
        HealthStatus status = new HealthStatus(
            "ok",
            Version.getVersion(),
            databaseStatus(),
            cacheStatus(),
            uptime()
        );
        return ResponseEntity.ok(status);
    }
}
```

**关键指标：**
- 错误率（目标：< 0.1%）
- 响应时间 p95（目标：< 200ms）
- 吞吐量（请求/秒）
- 资源使用率（CPU、内存、磁盘）
- 部署频率
- 变更失败率

## 常见合理化

| 合理化 | 现实 |
|---|---|
| "我们可以手动部署" | 手动部署不一致、易出错、不可重复。自动化消除人为错误。 |
| "CI 减慢了我们的速度" | 没有 CI 的速度是幻觉。你最终会花更多时间调试部署问题。 |
| "自动化太复杂" | 基本的 CI（测试 + 构建）很简单。复杂性来自不自动化。 |
| "我们稍后添加监控" | 没有监控的部署是盲目的。从第一天就开始监控。 |

## 危险信号

- 没有自动化测试的部署
- 手动部署步骤
- 不运行 lint 或类型检查的 CI
- 没有健康检查的部署
- 没有监控的生产环境
- 特性标志默认开启
- 没有回滚策略的部署

## 验证

设置 CI/CD 后：

- [ ] 每次 PR 自动运行测试
- [ ] lint 和代码检查在 CI 中运行
- [ ] 构建在部署前验证
- [ ] 集成测试在部署前运行
- [ ] 特性标志默认关闭
- [ ] 健康检查端点已配置
- [ ] 监控和警报已设置
- [ ] 回滚策略已测试