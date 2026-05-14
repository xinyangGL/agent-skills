---
name: security-and-hardening
description: 实施安全最佳实践并强化应用程序。当处理用户输入、认证、数据存储或外部集成时使用。当进行安全审查或加固应用程序时使用。当用户要求"使这个更安全"或"审查安全"时使用。
---

# 安全与加固

## 概览

实施安全最佳实践并强化应用程序。安全不是一次性检查——它是每一层架构中的持续纪律。 OWASP Top 10 是最低基线，不是目标。

## 何时使用

- 处理用户输入时
- 实现认证或授权时
- 存储或处理敏感数据时
- 集成外部服务或 API 时
- 进行安全审查时
- 用户要求加固应用程序时

## 三层边界系统

```
第 3 层（最外层）：  用户输入、请求、URL、查询参数
                    ↓ 验证 + 清理
第 2 层（中间层）：  已验证的数据、服务调用、数据库查询
                    ↓ 信任，但仍进行边界检查
第 1 层（最内层）：  核心业务逻辑、内部工具函数
                    ↓ 信任，无重复验证
```

**规则：** 只在边界处验证。内部代码信任已验证的数据。

## OWASP Top 10 预防

### 1. 注入（SQL、NoSQL、OS 命令、LDAP）

```java
// 好：参数化查询（JPA/QueryDSL）
public Task findByEmail(String email) {
    return taskRepository.findByEmail(email); // 参数化，安全
}

// 避免：字符串拼接（JPA/QueryDSL 中常见错误）
public Task findByEmailUnsafe(String email) {
    return entityManager.createQuery(
        "SELECT t FROM Task t WHERE email = '" + email + "'") // SQL 注入风险
        .getSingleResult();
}
```

**规则：** 永远不要将用户输入拼接到查询、命令或表达式中。

### 2. 认证失效

```java
// 好：强密码哈希（Spring Security）
@Bean
public PasswordEncoder passwordEncoder() {
    return new BCryptPasswordEncoder(12); // 12 轮，安全
}

// 好：会话安全（Spring Security）
http.sessionManagement(session -> session
    .sessionConcurrency(concurrency -> concurrency
        .maxSessions(1))
    .sessionFixation().migrateSession()
);
```

### 3. 敏感数据暴露

```java
// 好：从 API 响应中排除敏感字段
public record SafeUser(
    String id,
    String email,
    String name
) {}

public SafeUser toSafeUser(User user) {
    return new SafeUser(user.id(), user.email(), user.name());
}

// 好：环境变量中的密钥
@Value("${api.key}")
private String apiKey; // 不在代码中

// 避免：硬编码密钥
private static final String API_KEY = "sk-1234567890abcdef"; // 永远不要这样做
```

### 4. XSS（跨站脚本）

```java
// 好：Spring Boot 默认转义
@GetMapping("/tasks")
public String tasks(Model model) {
    model.addAttribute("taskTitle", userInput); // 默认转义，安全
    return "tasks";
}

// 避免：禁用转义
// <span th:text="${userInput}"></span> 默认转义
// <span th:utext="${userInput}"></span> 不转义，XSS 风险

// 如果必须使用 HTML，先清理
import org.owasp.html.PolicyFactory;
PolicyFactory policy = Sanitizers.FORMATTING.and(Sanitizers.LINKS);
String safeHtml = policy.sanitize(userInput);
```

### 5. 打破的访问控制

```java
// 好：在每个端点上检查授权
@GetMapping("/api/tasks/{id}")
@PreAuthorize("hasRole('USER')")
public ResponseEntity<Task> getTask(@PathVariable String id, Authentication auth) {
    Task task = taskService.findById(id);
    if (!task.getOwnerId().equals(auth.getName())) {
        return ResponseEntity.status(HttpStatus.FORBIDDEN).build();
    }
    return ResponseEntity.ok(task);
}
```

**规则：** 永远不要仅依赖前端隐藏来保护敏感操作。

### 6. 安全配置错误

```java
// 好：生产环境安全头部
@Configuration
public class SecurityConfig {
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http.headers(headers -> headers
            .contentSecurityPolicy(csp -> csp
                .policyDirectives("default-src 'self'; script-src 'self'"))
            .httpStrictTransportSecurity(hsts -> hsts
                .maxAgeInSeconds(31536000)
                .includeSubDomains(true))
            .frameOptions(frame -> frame.deny())
        );
        return http.build();
    }
}
```

### 7. 安全日志和监控

```java
// 好：安全的错误处理
@ControllerAdvice
public class GlobalExceptionHandler {
    @ExceptionHandler(Exception.class)
    public ResponseEntity<ApiError> handleException(Exception ex, HttpServletRequest request) {
        // 记录详细错误（内部）
        log.error("内部错误: {}", ex.getMessage(), ex);

        // 向用户返回通用消息
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR)
            .body(new ApiError(new Error("INTERNAL_ERROR", "出了点问题", null)));
    }
}
```

**规则：** 永远不要向用户暴露堆栈跟踪或内部详细信息。

## 密钥管理

```java
// 好：环境变量
@ConfigurationProperties(prefix = "app")
public class AppProperties {
    private String databaseUrl;
    private String apiKey;
    private String jwtSecret;
    // getters/setters
}

// 好：验证必需的密钥
@PostConstruct
public void validateSecrets() {
    if (appProperties.getJwtSecret() == null || appProperties.getJwtSecret().isBlank()) {
        throw new IllegalStateException("缺少必需的密钥: JWT_SECRET");
    }
}
```

**.env.example（提交到版本控制）：**
```
DATABASE_URL=postgresql://user:password@localhost:5432/dbname
API_KEY=your-api-key-here
JWT_SECRET=your-jwt-secret-here
```

## 依赖审计

```bash
# 检查已知漏洞
mvn dependency:check -DpluginId=owasp-dependency-check
# 或
./gradlew dependencyCheckAnalyze

# 自动修复（可能时）
mvn versions:use-latest-versions

# 检查严重漏洞
mvn dependency:check -DpluginId=owasp-dependency-check -DfailBuildOnCVSS=9
```

## 速率限制

```java
// 认证端点的速率限制
@Configuration
@EnableMethodSecurity
public class RateLimitConfig {
    @Bean
    public RateLimiter rateLimiter() {
        return RateLimiter.of(5, 15, TimeUnit.MINUTES); // 15 分钟内 5 次尝试
    }
}
```

## 常见合理化

| 合理化 | 现实 |
|---|---|
| "我们太小了，不是攻击目标" | 自动化攻击针对所有人，不分大小。安全是必需的。 |
| "我们稍后会添加安全" | 安全债务最难偿还。从第一天就开始安全。 |
| "框架处理了安全" | 框架防止了一些问题，但无法修复错误的配置或不安全的代码。 |
| "过度工程可以稍后进行" | 安全不是功能——它是基础。没有它，其他的一切都岌岌可危。 |

## 危险信号

- 硬编码的密钥或密码
- 向用户暴露的堆栈跟踪
- 未验证的用户输入
- 缺少速率限制的认证端点
- 没有 HTTPS
- 在日志中记录的敏感数据
- 缺少 CORS 配置
- `mvn dependency:check` 显示未修复的严重漏洞

## 验证

在安全审查完成后：

- [ ] 所有用户输入在边界处验证
- [ ] 密钥存储在环境变量中（不在代码中）
- [ ] 密码使用强算法哈希（BCrypt、Argon2）
- [ ] 会话 cookie 正确配置（httpOnly、secure、sameSite）
- [ ] 错误消息不暴露内部详细信息
- [ ] 安全头部已配置（CSP、HSTS 等）
- [ ] 认证端点有速率限制
- [ ] `mvn dependency:check` 没有严重或高危漏洞
- [ ] CORS 限制为特定来源