# 性能检查清单

Web 应用程序性能的快速参考检查清单。与 `performance-optimization` 技能配合使用。

## 目录

- [Core Web Vitals 目标](#core-web-vitals-目标)
- [TTFB 诊断](#ttfb-诊断)
- [前端检查清单](#前端检查清单)
- [后端检查清单](#后端检查清单)
- [测量命令](#测量命令)
- [常见反模式](#常见反模式)

## Core Web Vitals 目标

| 指标 | 良好 | 需要改进 | 较差 |
|------|------|----------|------|
| LCP (最大内容绘制) | ≤ 2.5s | ≤ 4.0s | > 4.0s |
| INP (交互到下一次绘制) | ≤ 200ms | ≤ 500ms | > 500ms |
| CLS (累积布局偏移) | ≤ 0.1 | ≤ 0.25 | > 0.25 |

## TTFB 诊断

当 TTFB 较慢（> 800ms）时，在 DevTools Network waterfall 中检查每个组件：

- [ ] **DNS 解析** 慢 → 为已知来源添加 `<link rel="dns-prefetch">` 或 `<link rel="preconnect">`
- [ ] **TCP/TLS 握手** 慢 → 启用 HTTP/2，考虑边缘部署，验证 keep-alive
- [ ] **服务器处理** 慢 → 分析后端，检查慢查询，添加缓存

## 前端检查清单

### 图片

- [ ] 图片使用现代格式（WebP、AVIF）
- [ ] 图片响应式大小（`srcset` 和 `sizes`）
- [ ] 图片和 `<source>` 元素有显式的 `width` 和 `height`（防止 CLS）
- [ ] 折叠下方的图片使用 `loading="lazy"` 和 `decoding="async"`
- [ ] Hero/LCP 图片使用 `fetchpriority="high"` 且不使用懒加载

### JavaScript

- [ ] Bundle 大小 gzip 后小于 200KB（初始加载）
- [ ] 使用动态 `import()` 进行代码分割（路由和重型功能）
- [ ] 启用 tree shaking（验证依赖提供 ESM 并标记 `sideEffects: false`）
- [ ] `<head>` 中没有阻塞 JavaScript（使用 `defer` 或 `async`）
- [ ] 重型计算卸载到 Web Workers（如适用）
- [ ] 在相同 props 下重新渲染的昂贵组件上使用 `React.memo()`
- [ ] 仅在性能分析显示有益时使用 `useMemo()` / `useCallback()`
- [ ] 长任务（> 50ms）被拆分以保持主线程可用 —— INP 的主要杠杆
- [ ] 在长循环中使用 `yieldToMain` 模式以便输入事件可以在块之间运行
- [ ] 在可用时使用现代调度 API：`scheduler.yield()`（推荐）、带优先级的 `scheduler.postTask()`、`isInputPending()` 仅在需要时 yield
- [ ] `requestIdleCallback` 用于可延迟的非紧急工作（分析刷新、预取、预热）
- [ ] 非关键工作推迟到事件处理器之外（例如分析、日志），以便对交互的响应不被延迟
- [ ] 第三方脚本使用 `async` / `defer` 加载、审计大小，并在重型时前面加 facade（聊天小部件、嵌入）

### CSS

- [ ] 关键 CSS 内联或预加载
- [ ] 非关键样式没有渲染阻塞 CSS
- [ ] 生产环境中没有 CSS-in-JS 运行时成本（使用提取）

### 字体

- [ ] 限制为 2-3 个字体家族，每个 2-3 个字重（每个额外字重是另一个请求）
- [ ] 仅 WOFF2 格式（最小，通用支持 —— 跳过 WOFF/TTF/EOT）
- [ ] 尽可能自托管（第三方字体 CDN 添加 DNS + TCP + TLS 往返）
- [ ] LCP 关键字体预加载：`<link rel="preload" as="font" type="font/woff2" crossorigin>`
- [ ] `font-display: swap`（或非关键的 `optional`）以避免 FOIT 阻塞渲染
- [ ] 通过 `unicode-range` 子集化，只发送每个页面需要的字形
- [ ] 当需要多个字重/样式时考虑可变字体（一个文件替换多个）
- [ ] 使用 `size-adjust`、`ascent-override`、`descent-override` 调整后备字体指标以减少字体替换时的 CLS
- [ ] 在任何自定义字体之前考虑系统字体栈

### 网络

- [ ] 静态资源使用长 `max-age` + 内容哈希缓存
- [ ] API 响应在适当时缓存（`Cache-Control`）
- [ ] 启用 HTTP/2 或 HTTP/3
- [ ] 已知来源资源预连接（`<link rel="preconnect">`）
- [ ] `fetchpriority` 用于关键非图片资源（例如关键的 `<link rel="preload">`、折叠上方的 `<script>`）—— 不仅用于 `<img>`
- [ ] 没有不必要的重定向

### 渲染

- [ ] 没有布局抖动（强制同步布局）
- [ ] 动画使用 `transform` 和 `opacity`（GPU 加速）
- [ ] 长列表使用虚拟化（例如 `react-window`）
- [ ] 没有不必要的全页面重新渲染
- [ ] 屏幕外区域使用 `content-visibility: auto` 和 `contain-intrinsic-size` 跳过非可见区域的布局/绘制
- [ ] 没有 `unload` 事件处理器，HTML 响应上没有 `Cache-Control: no-store` —— 保留回退/前进缓存（bfcache）资格

## 后端检查清单

### 数据库

- [ ] 没有 N+1 查询模式（使用预加载 / join）
- [ ] 查询有适当的索引
- [ ] 列表端点分页（从不 `SELECT * FROM table`）
- [ ] 连接池已配置
- [ ] 慢查询日志已启用

### API

- [ ] 响应时间 < 200ms（p95）
- [ ] 请求处理器中没有同步重型计算
- [ ] 批量操作而不是循环单个调用
- [ ] 响应压缩（gzip/brotli）
- [ ] 适当的缓存（内存、Redis、CDN）

### 基础设施

- [ ] 静态资源使用 CDN
- [ ] 服务器靠近用户（或边缘部署）
- [ ] 水平扩展已配置（如需要）
- [ ] 负载均衡器的健康检查端点

## 测量命令

### INP 现场数据和 DevTools 工作流

1. **先看现场数据** —— 在优化前检查 [CrUX Vis](https://developer.chrome.com/docs/crux/vis) 或你的 RUM 工具的真实用户 INP
2. **识别慢交互** —— 打开 DevTools → Performance 面板 → 交互时记录；查找由点击/按键触发的长任务
3. **在中端 Android 上测试** —— INP 问题通常只在慢硬件上出现；使用真实设备或 DevTools CPU 节流（4×-6× 减速）

```bash
# Lighthouse CLI
npx lighthouse https://localhost:3000 --output json --output-path ./report.json

# Bundle 分析
npx webpack-bundle-analyzer stats.json
# 或对于 Vite：
npx vite-bundle-visualizer

# 检查 bundle 大小
npx bundlesize

# 代码中的 Web Vitals
import { onLCP, onINP, onCLS } from 'web-vitals';
onLCP(console.log);
onINP(console.log);
onCLS(console.log);

# 带交互级别详细信息的 INP（归因构建）
import { onINP } from 'web-vitals/attribution';
onINP(({ value, attribution }) => {
  const { interactionTarget, inputDelay, processingDuration, presentationDelay } = attribution;
  console.log({ value, interactionTarget, inputDelay, processingDuration, presentationDelay });
});
```

## 常见反模式

| 反模式 | 影响 | 修复 |
|---|---|---|
| N+1 查询 | 数据库负载线性增长 | 使用 join、includes 或批量加载 |
| 无界查询 | 内存耗尽、超时 | 始终分页，添加 LIMIT |
| 缺少索引 | 数据增长时读取变慢 | 为过滤/排序的列添加索引 |
| 布局抖动 | 卡顿、掉帧 | 批量 DOM 读取，然后批量写入 |
| 图片未优化 | 慢 LCP、浪费带宽 | 使用 WebP、响应式大小、懒加载 |
| Bundle 过大 | 慢的可交互时间 | 代码分割、tree shaking、审计依赖 |
| 阻塞主线程 | 差 INP、UI 无响应 | 使用 `scheduler.yield()` / `yieldToMain` 分块长任务，卸载到 Web Workers |
| 内存泄漏 | 内存增长、最终崩溃 | 清理监听器、间隔、引用 |
