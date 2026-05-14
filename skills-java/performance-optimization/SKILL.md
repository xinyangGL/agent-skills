---
name: performance-optimization
description: 优化应用程序性能。当存在性能要求、怀疑性能回归、或需要改进 Core Web Vitals 或加载时间时使用。当分析揭示需要修复的瓶颈时使用。
---

# 性能优化

## 概览

优化前先测量。没有测量的性能工作是猜测——而猜测会导致过早优化，增加复杂性却没有改善真正重要的东西。先分析，找出实际瓶颈，修复它，再测量一次。只优化那些测量证明重要的部分。

## 何时使用

- 规格中存在性能要求（加载时间预算、响应时间 SLA）
- 用户或监控系统报告行为缓慢
- 响应时间超过阈值
- 你怀疑某个变更引入了性能回归
- 构建处理大型数据集或高流量的功能

**何时不要使用：** 在有问题证据之前不要优化。过早优化增加的复杂性比它带来的性能提升代价更大。

## 响应时间目标

| 层级 | 良好 | 需要改进 | 较差 |
|------|------|----------|------|
| API 响应 | < 200ms (p95) | < 500ms (p95) | > 500ms (p95) |
| 数据库查询 | < 50ms (p95) | < 200ms (p95) | > 200ms (p95) |
| 缓存命中 | < 10ms | < 50ms | > 50ms |

## 优化工作流

```
1. 测量  → 使用真实数据建立基线
2. 识别  → 找出实际瓶颈（而非假设的）
3. 修复  → 解决特定瓶颈
4. 验证  → 再次测量，确认改进
5. 防护  → 添加监控或测试以防止回归
```

### 第 1 步：测量

**后端：**
```java
// 响应时间日志
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

private static final Logger log = LoggerFactory.getLogger(TaskService.class);

public Task createTask(TaskRequest request) {
    long start = System.currentTimeMillis();
    Task task = taskRepository.save(new Task(request.getTitle()));
    long duration = System.currentTimeMillis() - start;
    log.debug("createTask 耗时: {}ms", duration);
    return task;
}
```

**数据库：**
```bash
# 慢查询日志（PostgreSQL）
# postgresql.conf
log_min_duration_statement = 100  # 日志记录超过 100ms 的查询

# 或 Spring Boot
spring.jpa.properties.hibernate.generate_statistics=true
```

### 第 2 步：识别瓶颈

**后端常见瓶颈：**

| 症状 | 可能原因 | 调查方法 |
|------|---------|---------|
| API 响应慢 | N+1 查询、缺少索引、未优化的查询 | 检查数据库查询日志 |
| 内存增长 | 泄漏的引用、无界缓存、大负载 | Heap dump 分析 |
| CPU 峰值 | 同步重型计算、正则回溯 | CPU profiling |
| 高延迟 | 缺少缓存、冗余计算、网络跳数 | 跟踪请求通过整个栈 |

### 第 3 步：修复常见反模式

#### N+1 查询（后端）

```java
// 差：N+1 — 每个任务一个查询获取所有者
List<Task> tasks = taskRepository.findAll();
for (Task task : tasks) {
    task.setOwner(userRepository.findById(task.getOwnerId()));
}

// 好：单个查询带 JOIN/Fetch
List<Task> tasks = taskRepository.findAllWithOwner();
```

#### 无界数据获取

```java
// 差：获取所有记录
List<Task> allTasks = taskRepository.findAll();

// 好：带限制的分页
Page<Task> tasks = taskRepository.findAll(PageRequest.of(page, size, Sort.by("createdAt").descending()));
```

#### 缺少缓存（后端）

```java
// 缓存频繁读取、很少更改的数据
@Cacheable(value = "appConfig", key = "'global'")
public AppConfig getAppConfig() {
    return configRepository.findFirstByType("GLOBAL")
        .orElseThrow(() -> new ConfigNotFoundException());
}
```

## 性能预算

```
API 响应时间：< 200ms（p95）
数据库查询：< 50ms（p95）
缓存命中：< 10ms
JVM 堆内存：< 2GB（生产环境）
GC 暂停：< 100ms
```

**在 CI 中强制执行：**
```bash
# 性能测试
mvn verify -P performance-tests
```

## 常见合理化

| 合理化 | 现实 |
|---|---|
| "我们之后再优化" | 性能债务会累积。现在就修复明显的反模式，延迟微观优化。 |
| "在我的机器上很快" | 你的机器不是用户的。在代表性硬件和网络上进行分析。 |
| "这个优化显而易见" | 如果你没有测量，你就不知道。先做分析。 |
| "用户不会注意到 100ms" | 研究表明 100ms 延迟影响转化率。用户注意到的比你以为的更多。 |
| "框架会处理性能" | 框架能防止一些问题，但无法修复 N+1 查询或过大的响应。 |

## 危险信号

- 没有性能分析数据支持的优化
- 数据获取中存在 N+1 查询模式
- 列表端点没有分页
- 缓存没有 TTL 或驱逐策略
- JVM 堆内存持续增长
- 生产环境中没有性能监控
- 过度使用 `@Synchronized` 或锁

## 验证

任何性能相关变更后：

- [ ] 存在变更前后的测量数据（具体数字）
- [ ] 已识别并解决特定瓶颈
- [ ] 响应时间在目标阈值范围内
- [ ] 没有引入 N+1 查询
- [ ] 性能测试在 CI 中通过（如果已配置）
- [ ] 现有测试仍然通过（优化没有破坏行为）