---
name: frontend-ui-engineering
description: 构建生产级 UI。当构建或修改面向用户的界面时使用。当创建组件、实现布局、管理状态、或输出需要看起来和感觉像生产级而非 AI 生成时使用。
---

# 前端 UI 工程

## 概览

构建生产级用户界面——可访问、高性能、视觉上精致。目标是看起来由顶级公司的设计意识工程师构建的 UI，而不是 AI 生成的。这意味着真正的设计系统遵循、适当的可访问性、深思熟虑的交互模式，没有通用的"AI 美学"。

## 何时使用

- 构建新的 UI 组件或页面
- 修改现有的面向用户的界面
- 实现响应式布局
- 添加交互性或状态管理
- 修复视觉或 UX 问题

## 组件架构

### 文件结构

将所有与组件相关的内容放在一起：

```
src/components/
  TaskList/
    TaskList.tsx          # 组件实现
    TaskList.test.tsx     # 测试
    TaskList.stories.tsx  # Storybook 故事（如果使用）
    use-task-list.ts      # 自定义 hook（如果状态复杂）
    types.ts              # 组件特定类型（如果需要）
```

### 组件模式

**优先使用组合而非配置：**

```tsx
// 好：可组合
<Card>
  <CardHeader>
    <CardTitle>任务</CardTitle>
  </CardHeader>
  <CardBody>
    <TaskList tasks={tasks} />
  </CardBody>
</Card>

// 避免：过度配置
<Card
  title="任务"
  headerVariant="large"
  bodyPadding="md"
  content={<TaskList tasks={tasks} />}
/>
```

**保持组件专注：**

```tsx
// 好：只做一件事
export function TaskItem({ task, onToggle, onDelete }: TaskItemProps) {
  return (
    <li className="flex items-center gap-3 p-3">
      <Checkbox checked={task.done} onChange={() => onToggle(task.id)} />
      <span className={task.done ? 'line-through text-muted' : ''}>{task.title}</span>
      <Button variant="ghost" size="sm" onClick={() => onDelete(task.id)}>
        <TrashIcon />
      </Button>
    </li>
  );
}
```

**分离数据获取和呈现：**

```tsx
// 容器：处理数据
export function TaskListContainer() {
  const { tasks, isLoading, error } = useTasks();

  if (isLoading) return <TaskListSkeleton />;
  if (error) return <ErrorState message="加载任务失败" retry={refetch} />;
  if (tasks.length === 0) return <EmptyState message="暂无任务" />;

  return <TaskList tasks={tasks} />;
}

// 呈现：处理渲染
export function TaskList({ tasks }: { tasks: Task[] }) {
  return (
    <ul role="list" className="divide-y">
      {tasks.map(task => <TaskItem key={task.id} task={task} />)}
    </ul>
  );
}
```

## 状态管理

**选择能工作的最简单方法：**

```
本地状态 (useState)          → 组件特定 UI 状态
提升状态                     → 2-3 个兄弟组件之间共享
Context                      → 主题、认证、语言（读多写少）
URL 状态 (searchParams)      → 过滤器、分页、可共享 UI 状态
服务器状态 (React Query, SWR)  → 带缓存的远程数据
全局存储 (Zustand, Redux)    → 全局共享的复杂客户端状态
```

**避免超过 3 层的 prop 钻取。** 如果你通过不使用它们的组件传递 props，引入 context 或重构组件树。

## 设计系统遵循

### 避免 AI 美学

AI 生成的 UI 有可识别的模式。避免所有这些问题：

| AI 默认值 | 为什么是问题 | 生产级质量 |
|---|---|---|
| 紫色/靛蓝 everywhere | 模型默认于视觉"安全"的调色板，使每个应用看起来都一样 | 使用项目实际的调色板 |
| 过多的渐变 | 渐变增加视觉噪声，与大多数设计系统冲突 | 匹配设计系统的平面或微妙渐变 |
| 所有都圆角 (rounded-2xl) | 最大圆角信号"友好"，但忽略了真实设计中圆角半径的层级 | 来自设计系统的一致 border-radius |
| 通用的 hero 部分 | 模板驱动的布局，与实际内容或用户需求没有联系 | 内容优先的布局 |
| Lorem ipsum 式文案 | 占位符文本隐藏了真实内容揭示的布局问题（长度、换行、溢出） | 现实的占位符内容 |
| 到处 oversized padding | 平等的慷慨 padding 破坏了视觉层次，浪费屏幕空间 | 一致的间距刻度 |
| 库存卡片网格 | 统一的网格是布局捷径，忽略了信息优先级和扫描模式 | 目的驱动的布局 |
| 阴影-heavy 设计 | 分层阴影增加深度，与内容竞争，在低端设备上减慢渲染 | 设计系统指定时才有微妙或无阴影 |

### 间距和布局

使用一致的间距刻度。不要发明值：

```css
/* 使用刻度：0.25rem 增量（或项目使用的任何值） */
/* 好 */  padding: 1rem;      /* 16px */
/* 好 */  gap: 0.75rem;       /* 12px */
/* 差 */  padding: 13px;      /* 不在任何刻度上 */
/* 差 */  margin-top: 2.3rem; /* 不在任何刻度上 */
```

### 排版

尊重类型层次：

```
h1 → 页面标题（每页一个）
h2 → 章节标题
h3 → 子章节标题
body → 默认文本
small → 次要/帮助文本
```

不要跳过标题级别。不要对非标题内容使用标题样式。

### 颜色

- 使用语义颜色令牌：`text-primary`、`bg-surface`、`border-default`——不是原始 hex 值
- 确保足够的对比度（正常文本 4.5:1，大文本 3:1）
- 不要仅依赖颜色来传达信息（也使用图标、文本或模式）

## 可访问性（WCAG 2.1 AA）

每个组件必须满足这些标准：

### 键盘导航

```tsx
// 每个交互元素必须可通过键盘访问
<button onClick={handleClick}>点击我</button>        // ✓ 默认可聚焦
<div onClick={handleClick}>点击我</div>               // ✗ 不可聚焦
```

### ARIA 标签

```tsx
// 标记缺少可见文本的交互元素
<button aria-label="关闭对话框"><XIcon /></button>

// 标记表单输入
<label htmlFor="email">邮箱</label>
<input id="email" type="email" />
```

### 焦点管理

```tsx
// 内容更改时移动焦点
function Dialog({ isOpen, onClose }: DialogProps) {
  const closeRef = useRef<HTMLButtonElement>(null);

  useEffect(() => {
    if (isOpen) closeRef.current?.focus();
  }, [isOpen]);

  return (
    <dialog open={isOpen}>
      <button ref={closeRef} onClick={onClose}>关闭</button>
      {/* 对话框内容 */}
    </dialog>
  );
}
```

### 有意义的空状态和错误状态

```tsx
// 不要显示空白屏幕
function TaskList({ tasks }: { tasks: Task[] }) {
  if (tasks.length === 0) {
    return (
      <div role="status" className="text-center py-12">
        <TasksEmptyIcon className="mx-auto h-12 w-12 text-muted" />
        <h3 className="mt-2 text-sm font-medium">暂无任务</h3>
        <p className="mt-1 text-sm text-muted">通过创建新任务开始。</p>
        <Button className="mt-4" onClick={onCreateTask}>创建任务</Button>
      </div>
    );
  }

  return <ul role="list">...</ul>;
}
```

## 响应式设计

从移动设备开始设计，然后扩展：

```tsx
// Tailwind：移动优先响应式
<div className="
  grid grid-cols-1      /* 移动：单列 */
  sm:grid-cols-2        /* 小屏：2 列 */
  lg:grid-cols-3        /* 大屏：3 列 */
  gap-4
">
```

在这些断点处测试：320px、768px、1024px、1440px。

## 加载和过渡

```tsx
// 骨架加载（内容不用加载 spinner）
function TaskListSkeleton() {
  return (
    <div className="space-y-3" aria-busy="true" aria-label="正在加载任务">
      {Array.from({ length: 3 }).map((_, i) => (
        <div key={i} className="h-12 bg-muted animate-pulse rounded" />
      ))}
    </div>
  );
}
```

## 常见合理化

| 合理化 | 现实 |
|---|---|
| "可访问性是锦上添花" | 在许多司法管辖区它是法律要求，也是工程质量标准。 |
| "我们之后再使其响应式" | 后加响应式设计比从头构建难 3 倍。 |
| "设计还没定，所以我跳过样式" | 使用设计系统默认值。无样式的 UI 会给审查者留下破碎的第一印象。 |
| "这只是一个原型" | 原型会变成生产代码。正确构建基础。 |
| "AI 美学现在没问题" | 它信号低质量。从一开始就使用项目实际的设计系统。 |

## 危险信号

- 超过 200 行的组件（拆分它们）
- 内联样式或任意像素值
- 缺少错误状态、加载状态或空状态
- 没有键盘导航测试
- 颜色作为状态的唯一指示器（没有文本或图标的红/绿）
- 通用的"AI 外观"（紫色渐变、超大卡片、库存布局）

## 验证

构建 UI 后：

- [ ] 组件渲染时无控制台错误
- [ ] 所有交互元素可通过键盘访问（Tab 遍历页面）
- [ ] 屏幕阅读器可以传达页面的内容和结构
- [ ] 响应式：在 320px、768px、1024px、1440px 下工作
- [ ] 处理了加载、错误和空状态
- [ ] 遵循项目的设计系统（间距、颜色、排版）
- [ ] 开发工具或 axe-core 中没有可访问性警告
