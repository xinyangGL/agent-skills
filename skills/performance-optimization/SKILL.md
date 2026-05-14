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
- Core Web Vitals 分数低于阈值
- 你怀疑某个变更引入了性能回归
- 构建处理大型数据集或高流量的功能

**何时不要使用：** 在有问题证据之前不要优化。过早优化增加的复杂性比它带来的性能提升代价更大。

## Core Web Vitals 目标

| 指标 | 良好 | 需要改进 | 较差 |
|------|------|----------|------|
| **LCP** (最大内容绘制) | ≤ 2.5s | ≤ 4.0s | > 4.0s |
| **INP** (交互到下一次绘制) | ≤ 200ms | ≤ 500ms | > 500ms |
| **CLS** (累积布局偏移) | ≤ 0.1 | ≤ 0.25 | > 0.25 |

## 优化工作流

```
1. 测量  → 使用真实数据建立基线
2. 识别  → 找出实际瓶颈（而非假设的）
3. 修复  → 解决特定瓶颈
4. 验证  → 再次测量，确认改进
5. 防护  → 添加监控或测试以防止回归
```

### 第 1 步：测量

**前端：**
```bash
# 合成测试：Chrome DevTools 中的 Lighthouse
# Chrome DevTools → Performance 面板 → 记录

# RUM：代码中的 Web Vitals 库
import { onLCP, onINP, onCLS } from 'web-vitals';
onLCP(console.log);
onINP(console.log);
onCLS(console.log);
```

**后端：**
```bash
# 响应时间日志
console.time('db-query');
const result = await db.query(...);
console.timeEnd('db-query');
```

### 第 2 步：识别瓶颈

**前端常见瓶颈：**

| 症状 | 可能原因 | 调查方法 |
|------|---------|---------|
| LCP 慢 | 大图片、渲染阻塞资源、服务器慢 | 检查 network waterfall、图片大小 |
| CLS 高 | 无尺寸的图片、延迟加载内容、字体偏移 | 检查布局偏移归因 |
| INP 差 | 主线程上的重型 JavaScript、大型 DOM 更新 | 在 Performance trace 中检查长任务 |

**后端常见瓶颈：**

| 症状 | 可能原因 | 调查方法 |
|------|---------|---------|
| API 响应慢 | N+1 查询、缺少索引、未优化的查询 | 检查数据库查询日志 |
| 内存增长 | 泄漏的引用、无界缓存、大负载 | Heap snapshot 分析 |
| CPU 峰值 | 同步重型计算、正则回溯 | CPU 性能分析 |

### 第 3 步：修复常见反模式

#### N+1 查询（后端）

```typescript
// 差：N+1 — 每个任务一个查询获取所有者
const tasks = await db.tasks.findMany();
for (const task of tasks) {
  task.owner = await db.users.findUnique({ where: { id: task.ownerId } });
}

// 好：单个查询带 join/include
const tasks = await db.tasks.findMany({
  include: { owner: true },
});
```

#### 无界数据获取

```typescript
// 差：获取所有记录
const allTasks = await db.tasks.findMany();

// 好：带限制的分页
const tasks = await db.tasks.findMany({
  take: 20,
  skip: (page - 1) * 20,
  orderBy: { createdAt: 'desc' },
});
```

#### 不必要的重新渲染（React）

```tsx
// 差：每次渲染创建新对象，导致子组件重新渲染
function TaskList() {
  return <TaskFilters options={{ sortBy: 'date', order: 'desc' }} />;
}

// 好：稳定的引用
const DEFAULT_OPTIONS = { sortBy: 'date', order: 'desc' } as const;
function TaskList() {
  return <TaskFilters options={DEFAULT_OPTIONS} />;
}
```

#### 缺少缓存（后端）

```typescript
// 缓存频繁读取、很少更改的数据
const CACHE_TTL = 5 * 60 * 1000; // 5 分钟
let cachedConfig: AppConfig | null = null;
let cacheExpiry = 0;

async function getAppConfig(): Promise<AppConfig> {
  if (cachedConfig && Date.now() < cacheExpiry) {
    return cachedConfig;
  }
  cachedConfig = await db.config.findFirst();
  cacheExpiry = Date.now() + CACHE_TTL;
  return cachedConfig;
}
```

## 性能预算

```
JavaScript bundle：初始加载 gzip 后 < 200KB
CSS：gzip 后 < 50KB
图片：折叠上方每张图片 < 200KB
字体：总计 < 100KB
API 响应时间：< 200ms（p95）
可交互时间：4G 网络下 < 3.5s
Lighthouse 性能分数：≥ 90
```

**在 CI 中强制执行：**
```bash
# Bundle 大小检查
npx bundlesize --config bundlesize.config.json

# Lighthouse CI
npx lhci autorun
```

## 常见合理化

| 合理化 | 现实 |
|---|---|
| "我们之后再优化" | 性能债务会累积。现在就修复明显的反模式，延迟微观优化。 |
| "在我的机器上很快" | 你的机器不是用户的。在代表性硬件和网络上进行分析。 |
| "这个优化显而易见" | 如果你没有测量，你就不知道。先做分析。 |
| "用户不会注意到 100ms" | 研究表明 100ms 延迟影响转化率。用户注意到的比你以为的更多。 |
| "框架会处理性能" | 框架能防止一些问题，但无法修复 N+1 查询或过大的 bundle。 |

## 危险信号

- 没有性能分析数据支持的优化
- 数据获取中存在 N+1 查询模式
- 列表端点没有分页
- 图片没有尺寸、懒加载或响应式大小
- Bundle 大小在没有审查的情况下增长
- 生产环境中没有性能监控
- 到处使用 `React.memo` 和 `useMemo`（过度使用和不足使用一样糟糕）

## 验证

任何性能相关变更后：

- [ ] 存在变更前后的测量数据（具体数字）
- [ ] 已识别并解决特定瓶颈
- [ ] Core Web Vitals 在"良好"阈值范围内
- [ ] Bundle 大小没有显著增加
- [ ] 新的数据获取代码中没有 N+1 查询
- [ ] 性能预算在 CI 中通过（如果已配置）
- [ ] 现有测试仍然通过（优化没有破坏行为）
