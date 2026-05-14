# 安全检查清单

Web 应用程序安全的快速参考。与 `security-and-hardening` 技能配合使用。

## 目录

- [提交前检查](#提交前检查)
- [认证](#认证)
- [授权](#授权)
- [输入验证](#输入验证)
- [安全头部](#安全头部)
- [CORS 配置](#cors-配置)
- [数据保护](#数据保护)
- [依赖安全](#依赖安全)
- [错误处理](#错误处理)
- [OWASP Top 10 快速参考](#owasp-top-10-快速参考)

## 提交前检查

- [ ] 代码中没有密钥（`git diff --cached | grep -i "password\|secret\|api_key\|token"`）
- [ ] `.gitignore` 覆盖了：`.env`、`.env.local`、`*.pem`、`*.key`
- [ ] `.env.example` 使用占位值（不是真实密钥）

## 认证

- [ ] 密码使用 bcrypt（≥12 轮）、scrypt 或 argon2 哈希
- [ ] 会话 cookie：`httpOnly`、`secure`、`sameSite: 'lax'`
- [ ] 会话过期已配置（合理的 max-age）
- [ ] 登录端点速率限制（≤10 次尝试/15 分钟）
- [ ] 密码重置令牌：限时（≤1 小时）、一次性使用
- [ ] 重复失败后账户锁定（可选，带通知）
- [ ] 敏感操作支持 MFA（可选但推荐）

## 授权

- [ ] 每个受保护端点检查认证
- [ ] 每个资源访问检查所有权/角色（防止 IDOR）
- [ ] 管理员端点要求管理员角色验证
- [ ] API 密钥范围限制在最小必要权限
- [ ] JWT 令牌已验证（签名、过期、签发者）

## 输入验证

- [ ] 所有用户输入在系统边界处验证（API 路由、表单处理器）
- [ ] 验证使用白名单（不是黑名单）
- [ ] 字符串长度受限（最小/最大）
- [ ] 数字范围已验证
- [ ] 邮箱、URL 和日期格式使用适当的库验证
- [ ] 文件上传：类型受限、大小限制、内容验证
- [ ] SQL 查询参数化（不拼接字符串）
- [ ] HTML 输出已编码（使用框架自动转义）
- [ ] 重定向前验证 URL（防止开放重定向）

## 安全头部

```
Content-Security-Policy: default-src 'self'; script-src 'self'
Strict-Transport-Security: max-age=31536000; includeSubDomains
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
X-XSS-Protection: 0  (禁用，依赖 CSP)
Referrer-Policy: strict-origin-when-cross-origin
Permissions-Policy: camera=(), microphone=(), geolocation=()
```

## CORS 配置

```typescript
// 严格（推荐）
cors({
  origin: ['https://yourdomain.com', 'https://app.yourdomain.com'],
  credentials: true,
  methods: ['GET', 'POST', 'PUT', 'PATCH', 'DELETE'],
  allowedHeaders: ['Content-Type', 'Authorization'],
})

// 绝不在生产环境使用：
cors({ origin: '*' })  // 允许任何来源
```

## 数据保护

- [ ] 敏感字段从 API 响应中排除（`passwordHash`、`resetToken` 等）
- [ ] 敏感数据不记录日志（密码、令牌、完整信用卡号）
- [ ] PII 静态加密（如法规要求）
- [ ] 所有外部通信使用 HTTPS
- [ ] 数据库备份已加密

## 依赖安全

```bash
# 审计依赖
npm audit

# 自动修复（可能时）
npm audit fix

# 检查严重漏洞
npm audit --audit-level=critical

# 保持依赖更新
npx npm-check-updates
```

## 错误处理

```typescript
// 生产环境：通用错误，不暴露内部信息
res.status(500).json({
  error: { code: 'INTERNAL_ERROR', message: '出了点问题' }
});

// 生产环境中绝不要：
res.status(500).json({
  error: err.message,
  stack: err.stack,         // 暴露内部信息
  query: err.sql,           // 暴露数据库细节
});
```

## OWASP Top 10 快速参考

| # | 漏洞 | 预防 |
|---|---|---|
| 1 | 访问控制失效 | 每个端点检查认证，所有权验证 |
| 2 | 加密失败 | HTTPS、强哈希、代码中无密钥 |
| 3 | 注入 | 参数化查询、输入验证 |
| 4 | 不安全设计 | 威胁建模、规格驱动开发 |
| 5 | 安全配置错误 | 安全头部、最小权限、审计依赖 |
| 6 | 易受攻击的组件 | `npm audit`、保持依赖更新、最小依赖 |
| 7 | 认证失败 | 强密码、速率限制、会话管理 |
| 8 | 数据完整性失败 | 验证更新/依赖、签名制品 |
| 9 | 日志和监控失败 | 记录安全事件，不记录密钥 |
| 10 | SSRF | 验证/白名单 URL、限制出站请求 |
