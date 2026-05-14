---
name: browser-testing-with-devtools
description: 通过实时浏览器测试验证应用程序。当构建或调试任何在浏览器中运行的内容时使用。当你在构建过程中或行为在静态代码审查中不清楚时需要运行时数据时使用。
---

# 浏览器测试与 DevTools

## 概览

使用 Chrome DevTools MCP 进行真实的运行时浏览器测试——不是截图，不是静态验证。验证你的应用程序在浏览器中实际运行。

## 何时使用

- 构建任何在浏览器中运行的内容
- 调试在本地通过但在浏览器中失败的测试
- 需要运行时数据（DOM 状态、网络请求、控制台输出）
- 验证 UI 交互、表单提交或页面导航
- 行为在静态代码审查中不清楚

## 设置

```bash
# 启动 Chrome DevTools MCP
npx @anthropic/chrome-devtools-mcp-server
```

## 工作流

```
1. 导航 → 加载你要测试的页面
2. 交互 → 像用户一样点击、输入、选择
3. 观察 → 检查 DOM、控制台、网络、性能
4. 断言 → 验证状态匹配预期
5. 报告 → 记录结果和发现的问题
```

### 第 1 步：导航

```typescript
// 导航到测试页面
await page.goto('http://localhost:3000/tasks');

// 或导航到特定路由
await page.goto('http://localhost:3000/tasks/123');
```

### 第 2 步：交互

```typescript
// 点击按钮
await page.click('[data-testid="new-task-button"]');

// 输入文本
await page.fill('[data-testid="task-title"]', '新任务');

// 选择下拉选项
await page.selectOption('[data-testid="priority-select"]', 'high');

// 提交表单
await page.click('[data-testid="submit-button"]');
```

### 第 3 步：观察

```typescript
// 检查 DOM 状态
const text = await page.locator('[data-testid="task-list"]').textContent();
console.log('任务列表内容:', text);

// 检查控制台日志
const logs = await page.console();
console.log('控制台输出:', logs);

// 检查网络请求
const requests = await page.network();
console.log('网络请求:', requests);

// 检查页面标题
const title = await page.title();
console.log('页面标题:', title);
```

### 第 4 步：断言

```typescript
// 验证元素存在
await expect(page.locator('[data-testid="task-item"]')).toBeVisible();

// 验证文本内容
await expect(page.locator('[data-testid="task-title"]')).toHaveText('新任务');

// 验证 URL
await expect(page).toHaveURL('http://localhost:3000/tasks/123');

// 验证控制台没有错误
const errors = await page.console({ type: 'error' });
expect(errors.length).toBe(0);
```

### 第 5 步：报告

```typescript
// 截图用于文档
await page.screenshot({ path: 'test-results/task-creation.png' });

// 记录结果
console.log('测试通过：任务创建工作流程正常');
console.log('最终 URL:', page.url());
console.log('任务计数:', await page.locator('[data-testid="task-item"]').count());
```

## 常见测试场景

### 表单提交

```typescript
test('创建任务表单', async ({ page }) => {
  await page.goto('http://localhost:3000/tasks');

  // 填写表单
  await page.fill('[data-testid="task-title"]', '测试任务');
  await page.fill('[data-testid="task-description"]', '测试描述');
  await page.selectOption('[data-testid="priority-select"]', 'high');

  // 提交
  await page.click('[data-testid="submit-button"]');

  // 验证结果
  await expect(page.locator('[data-testid="task-item"]')).toBeVisible();
  await expect(page.locator('[data-testid="task-title"]')).toHaveText('测试任务');

  // 验证没有控制台错误
  const errors = await page.console({ type: 'error' });
  expect(errors.length).toBe(0);
});
```

### 页面导航

```typescript
test('导航到任务详情', async ({ page }) => {
  await page.goto('http://localhost:3000/tasks');

  // 点击第一个任务
  await page.click('[data-testid="task-item"] >> nth=0');

  // 验证导航
  await expect(page).toHaveURL(/\/tasks\/\d+/);
  await expect(page.locator('[data-testid="task-detail"]')).toBeVisible();
});
```

### 键盘交互

```typescript
test('键盘导航', async ({ page }) => {
  await page.goto('http://localhost:3000/tasks');

  // Tab 到第一个可交互元素
  await page.keyboard.press('Tab');
  const focused = await page.evaluate(() => document.activeElement?.tagName);
  expect(focused).toBe('BUTTON');

  // Enter 激活按钮
  await page.keyboard.press('Enter');

  // 验证模态框打开
  await expect(page.locator('[data-testid="task-modal"]')).toBeVisible();
});
```

## 常见合理化

| 合理化 | 现实 |
|---|---|
| "单元测试就够了" | 单元测试不测试浏览器行为。CSS、JavaScript 互操作性、和 DOM 操作用于单元不可见。 |
| "E2E 太慢了" | 有针对性的浏览器测试比完整的 E2E 套件快。测试关键路径，不是所有路径。 |
| "我可以在本地手动测试" | 手动测试不可重复。自动化浏览器测试在 CI 中运行，每次部署时运行。 |
| "这个组件太简单了" | 即使是简单的组件也会在浏览器中以意想不到的方式失败。测试它。 |

## 危险信号

- 不在浏览器中测试浏览器代码
- 依赖截图作为唯一的验证
- 不在 CI 中自动化浏览器测试
- 测试实现细节而不是用户可见的行为
- 不检查控制台错误
- 不测试键盘导航（可访问性）

## 验证

浏览器测试后：

- [ ] 测试页面在浏览器中正确加载
- [ ] 交互（点击、输入、选择）按预期工作
- [ ] DOM 状态匹配预期
- [ ] 没有控制台错误
- [ ] 网络请求成功
- [ ] 键盘导航工作（可访问性）
- [ ] 测试结果已记录
