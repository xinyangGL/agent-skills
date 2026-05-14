---
name: code-simplification
description: 在不改变行为的情况下简化和澄清代码。当代码能运行但比应有的更难阅读或维护时使用。当重构遗留代码、清理技术债务或改进可读性时使用。当用户要求"简化这个"或"清理这个代码"时使用。
---

# 代码简化

## 概览

在不改变行为的情况下简化代码。目标是通过降低复杂度和提高清晰度来使代码更易于理解、维护和修改。好的简化保留精确行为，同时使意图显而易见的。

## 何时使用

- 代码能运行但比应有的更难阅读时
- 重构遗留代码时
- 清理技术债务时
- 用户要求简化或清理代码时
- 代码有深层嵌套、长函数或重复模式时

## 何时不使用

- 代码已经清晰且简洁时
- 用户明确要求快速修复且接受不简化的权衡时
- 在不稳定或正在积极开发的代码上（先稳定它）

## Chesterton 栅栏

在移除任何东西之前，了解它为什么存在。

```
看到看似不必要的代码？
1. 尝试理解它为什么在这里
2. 查找处理边缘情况的证据
3. 检查相关问题和提交历史
4. 只有在你理解它为什么存在后才能移除它
```

**规则：** 不要移除你不理解为什么存在的东西。

## 500 规则

如果解释代码如何工作的注释超过 500 个字，代码可能需要简化。

```typescript
// 如果你需要这样的注释来解释代码：
/*
 * 这个函数处理用户认证，但只有在用户不是管理员且
 * 会话没有过期且请求来自受信任的域且...
 */
function authenticateUser(user, session, request) {
  // 50 行复杂的逻辑
}

// 考虑将其拆分为更清晰的函数：
function isAuthenticationRequired(user, request) { ... }
function isValidSession(session) { ... }
function isTrustedDomain(request) { ... }
function authenticateUser(user, session, request) {
  if (!isAuthenticationRequired(user, request)) return;
  if (!isValidSession(session)) throw new SessionExpiredError();
  if (!isTrustedDomain(request)) throw new UnauthorizedError();
  // 认证逻辑
}
```

## 简化过程

### 第 1 步：理解当前行为

在更改任何内容之前，彻底理解代码做什么。

```markdown
1. 阅读代码和相关测试
2. 运行测试以确认它们通过
3. 写下你对代码行为的理解
4. 识别输入、输出和副作用
```

### 第 2 步：识别问题

找到使代码难以阅读或维护的模式。

```markdown
常见反模式：
- 深层嵌套（> 3 层）
- 长函数（> 50 行）
- 重复代码（DRY 违反）
- 魔法数字和字符串
- 不清晰的命名
- 过多的职责
```

### 第 3 步：应用简化技术

按优先级应用简化：

**1. 提取函数**
```typescript
// 之前：一个长函数
function processOrder(order) {
  // 验证
  if (!order.items) throw new Error('...');
  if (order.items.length === 0) throw new Error('...');
  // 计算
  let total = 0;
  for (const item of order.items) {
    total += item.price * item.quantity;
  }
  // 应用折扣
  if (order.discount) total *= (1 - order.discount);
  // 保存
  db.orders.save({ ...order, total });
}

// 之后：提取的函数
function validateOrder(order) { ... }
function calculateTotal(order) { ... }
function applyDiscount(total, discount) { ... }
function processOrder(order) {
  validateOrder(order);
  const total = applyDiscount(calculateTotal(order), order.discount);
  db.orders.save({ ...order, total });
}
```

**2. 提前返回（守卫子句）**
```typescript
// 之前：深层嵌套
function getUser(id) {
  if (id) {
    const user = db.users.find(id);
    if (user) {
      if (user.active) {
        return user;
      }
    }
  }
  return null;
}

// 之后：守卫子句
function getUser(id) {
  if (!id) return null;
  const user = db.users.find(id);
  if (!user) return null;
  if (!user.active) return null;
  return user;
}
```

**3. 使用有意义的名称**
```typescript
// 之前
const d = new Date();
const y = d.getFullYear();

// 之后
const today = new Date();
const currentYear = today.getFullYear();
```

### 第 4 步：验证行为未改变

确保简化没有改变行为。

```markdown
1. 运行所有测试（应该仍然通过）
2. 手动测试关键场景（如适用）
3. 比较简化前后的输出
4. 审查变更以确保行为一致
```

## 简化检查清单

- [ ] 理解当前行为（测试通过）
- [ ] 识别问题模式
- [ ] 应用简化技术
- [ ] 验证行为未改变（测试仍然通过）
- [ ] 代码比之前更清晰
- [ ] 没有引入新的复杂性

## 常见合理化

| 合理化 | 现实 |
|---|---|
| "它能工作，不要修复没坏的东西" | 难以维护的代码最终会坏。简化防止未来 bug。 |
| "重构可以稍后进行" | 技术债务累积利息。越早简化，成本越低。 |
| "这个代码很复杂，因为它处理的的问题很复杂" | 复杂的问题需要清晰的代码，而不是复杂的代码。 |
| "我会理解它并重命名" | 如果你需要理解它才能重命名，你需要理解它才能简化。 |

## 危险信号

- 在理解当前行为前更改代码
- 简化后测试失败
- 引入新的抽象而不增加清晰度
- 重命名为了使代码"更酷"而不是更清晰
- 移除你不理解为什么存在的东西（Chesterton 栅栏）

## 验证

在简化完成后：

- [ ] 所有测试通过（行为未改变）
- [ ] 代码比之前更清晰
- [ ] 没有引入新的复杂性
- [ ] 命名具有描述性且一致
- [ ] 函数短小且专注
- [ ] 嵌套最小化
- [ ] 没有重复代码
