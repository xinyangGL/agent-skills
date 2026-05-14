# 无障碍检查清单

WCAG 2.1 AA 合规的快速参考。与 `frontend-ui-engineering` 技能配合使用。

## 目录

- [基本检查](#基本检查)
- [常见 HTML 模式](#常见-html-模式)
- [测试工具](#测试工具)
- [快速参考：ARIA 实时区域](#快速参考aria-实时区域)
- [常见反模式](#常见反模式)

## 基本检查

### 键盘导航

- [ ] 所有交互元素可通过 Tab 键聚焦
- [ ] 焦点顺序遵循视觉/逻辑顺序
- [ ] 焦点可见（聚焦元素上有 outline/ring）
- [ ] 自定义组件有键盘支持（Enter 激活、Escape 关闭）
- [ ] 没有键盘陷阱（用户始终可以 Tab 离开组件）
- [ ] 页面顶部有跳转到内容链接 —— 至少在键盘聚焦时可见
- [ ] 模态框在打开时捕获焦点，关闭时返回焦点

### 屏幕阅读器

- [ ] 所有图片有 `alt` 文本（装饰性图片使用 `alt=""`）
- [ ] 所有表单输入有关联标签（`<label>` 或 `aria-label`）
- [ ] 按钮和链接有描述性文本（不是"点击这里"）
- [ ] 仅图标的按钮有 `aria-label`
- [ ] 页面有一个 `<h1>` 且标题不跳过级别
- [ ] 动态内容变更已宣布（`aria-live` 区域）
- [ ] 表格有 `<th>` 表头和 scope

### 视觉

- [ ] 文本对比度 ≥ 4.5:1（正常文本）或 ≥ 3:1（大文本，18px+）
- [ ] UI 组件对比度 ≥ 3:1（与背景）
- [ ] 颜色不是传达信息的唯一方式
- [ ] 文本可调整到 200% 且不破坏布局
- [ ] 没有每秒闪烁超过 3 次的

## 常见 HTML 模式

### 模态框

```html
<div role="dialog" aria-modal="true" aria-labelledby="modal-title">
  <h2 id="modal-title">编辑任务</h2>
  <!-- 模态框内容 -->
  <button aria-label="关闭模态框" onclick="closeModal()">关闭</button>
</div>
```

### 导航

```html
<nav aria-label="主导航">
  <ul>
    <li><a href="/" aria-current="page">首页</a></li>
    <li><a href="/tasks">任务</a></li>
    <li><a href="/about">关于</a></li>
  </ul>
</nav>
```

### 表单

```html
<form>
  <label for="email">邮箱</label>
  <input type="email" id="email" required aria-required="true">

  <label for="password">密码</label>
  <input type="password" id="password" required aria-required="true"
         aria-describedby="password-hint">
  <div id="password-hint">密码至少 8 个字符</div>

  <button type="submit">提交</button>
</form>
```

### 下拉菜单

```html
<div class="dropdown">
  <button aria-haspopup="true" aria-expanded="false">选项</button>
  <ul hidden>
    <li><button>选项 1</button></li>
    <li><button>选项 2</button></li>
  </ul>
</div>
```

### 标签页

```html
<div role="tablist" aria-label="任务筛选">
  <button role="tab" aria-selected="true" aria-controls="tab-all">全部</button>
  <button role="tab" aria-selected="false" aria-controls="tab-active">进行中</button>
  <button role="tab" aria-selected="false" aria-controls="tab-done">已完成</button>
</div>
<div role="tabpanel" id="tab-all">...</div>
<div role="tabpanel" id="tab-active" hidden>...</div>
<div role="tabpanel" id="tab-done" hidden>...</div>
```

### 下拉选择

```html
<select name="priority" id="priority">
  <option value="">选择优先级</option>
  <option value="low">低</option>
  <option value="medium">中</option>
  <option value="high">高</option>
</select>
```

### 通知/警报

```html
<div role="alert">任务已创建</div>
<div role="status" aria-live="polite">正在保存...</div>
```

### 工具提示

```html
<button aria-describedby="tip-delete">删除</button>
<div role="tooltip" id="tip-delete" hidden>
  此操作不可撤销
</div>
```

### 图片

```html
<!-- 信息性图片 -->
<img src="chart.png" alt="2024 年任务完成趋势图，显示稳步增长">

<!-- 装饰性图片 -->
<img src="divider.png" alt="" role="presentation">

<!-- 复杂图片（需要描述） -->
<figure>
  <img src="architecture.png" alt="系统架构图">
  <figcaption>系统架构：客户端通过 API 网关与微服务通信。</figcaption>
</figure>
```

### 视频/音频

```html
<video controls poster="thumbnail.jpg">
  <source src="video.mp4" type="video/mp4">
  <track src="captions.vtt" kind="captions" srclang="zh" label="中文">
  <p>您的浏览器不支持视频播放。</p>
</video>
```

## 测试工具

### 手动测试

- [ ] **键盘导航：** 只用键盘（Tab、Shift+Tab、Enter、Space、Arrow keys）导航整个页面
- [ ] **屏幕阅读器：** 使用 NVDA（Windows）、VoiceOver（macOS）或 TalkBack（Android）
- [ ] **屏幕放大：** 将浏览器缩放至 200%，检查布局是否保持可用
- [ ] **移动设备：** 使用 VoiceOver（iOS）或 TalkBack（Android）测试触摸交互

### 自动化工具

| 工具 | 类型 | 用法 |
|------|------|------|
| **axe DevTools** | 浏览器扩展 | 自动检测可访问性违规 |
| **Lighthouse** | Chrome DevTools | 性能 + 可访问性审计 |
| **WAVE** | 浏览器扩展 | 可视可访问性评估 |
| **Google Search Console** | 在线工具 | 检查移动可用性和可访问性 |
| **contrastchecker.com** | 在线工具 | 验证颜色对比度 |

### CI 中的自动化

```bash
# axe CLI
npx axe-cli https://localhost:3000

# Lighthouse 可访问性审计
npx lighthouse https://localhost:3000 --only-categories=accessibility

# Playwright 可访问性测试
npx playwright test --grep accessibility
```

## 快速参考：ARIA 实时区域

| 属性 | 用途 | 示例 |
|------|------|------|
| `aria-live="polite"` | 在用户空闲时宣布 | 状态更新、计数变化 |
| `aria-live="assertive"` | 立即宣布，打断当前内容 | 错误、警报 |
| `aria-live="off"` | 不宣布 | 装饰性动态内容 |
| `aria-atomic="true"` | 宣布整个区域（不是仅变更部分） | 状态消息 |
| `aria-relevant="additions removals"` | 仅宣布添加/移除 | 列表更新 |

## 常见反模式

| 反模式 | 问题 | 修复 |
|--------|------|------|
| `tabindex="0"` 在不可交互元素上 | 创建混乱的焦点顺序 | 仅对可交互元素使用 `tabindex="0"` |
| `tabindex="-1"` 在不可聚焦元素上 | 跳过键盘导航 | 移除 `tabindex` 或使元素可交互 |
| `<div>` 或 `<span>` 作为按钮 | 没有键盘支持或屏幕阅读器语义 | 使用 `<button>` |
| `<a>` 用于非导航 | 链接语义是"去某处"，不是"做某事" | 使用 `<button>` |
| 装饰性图片有 `alt` 文本 | 向屏幕阅读器用户朗读不必要的信息 | 使用 `alt=""` |
| 信息性图片没有 `alt` 文本 | 屏幕阅读器跳过图片 | 添加描述性 `alt` 文本 |
| 隐藏元素没有 `display: none` | 屏幕阅读器仍然读取它们 | 使用 `display: none`（不是仅 `visibility: hidden`） |
| 没有 `aria-label` 的图标按钮 | 屏幕阅读器朗读 SVG 标签名称（"path"、"polygon"） | 添加 `aria-label` |
| 颜色作为唯一指示器 | 色盲用户无法区分 | 添加图标、文本或模式 |
| 焦点陷阱在模态框外 | 用户在模态框打开时可以 Tab 到后台内容 | 在模态框中捕获焦点 |
