# Java 项目配置模板 (CLAUDE.md)

# 项目：[项目名称]

## 技术栈
- Java 17, Spring Boot 3, Maven, PostgreSQL, Flyway
- JUnit 5, Mockito, Testcontainers
- Spring Security, Lombok（可选）

## 命令
- 构建：`mvn clean package`
- 测试：`mvn test`
- Lint：`mvn spotless:apply`
- 开发：`mvn spring-boot:run`
- 类型检查：`mvn compile`
- 依赖审计：`mvn dependency:check -DpluginId=owasp-dependency-check`

## 代码约定
- 遵循 Google Java Style Guide
- 使用 Record 类型作为 DTO（Java 16+）
- 测试与源码共存：`TaskService.java` → `TaskServiceTest.java`
- 使用 Spring Security 处理认证和授权
- 使用 JPA/Hibernate 进行数据访问
- 使用 Bean Validation 注解（@Valid, @NotBlank 等）在边界处验证输入
- 使用 SLF4J 日志（log.debug()）

## 项目结构
```
src/main/java/
  com/example/app/
    controller/     # REST 控制器
    service/        # 业务逻辑
    repository/     # 数据访问
    model/          # 实体和 DTO
    config/         # 配置类
    exception/      # 异常处理
src/test/java/
  com/example/app/
    controller/     # 控制器测试（MockMvc）
    service/        # 服务测试
    integration/    # 集成测试（Testcontainers）
```

## 测试策略
- 单元测试：所有 Service 层方法（80%+ 覆盖）
- 集成测试：所有 Controller 端点（MockMvc + Testcontainers）
- E2E 测试：关键用户流程（Selenium/Playwright）

## 边界
- 绝不提交 .env 文件或密钥
- 绝不添加依赖而不检查包大小影响
- 修改数据库模式前询问
- 提交前始终运行测试
- 始终使用参数化查询（防止 SQL 注入）
- 始终使用 BCrypt 哈希密码
- 始终在 API 边界处验证输入（@Valid）

## 模式示例
```java
@RestController
@RequestMapping("/api/tasks")
public class TaskController {

    @PostMapping
    public ResponseEntity<Task> createTask(
            @Valid @RequestBody CreateTaskInput input) {
        Task task = taskService.create(input);
        return ResponseEntity.status(HttpStatus.CREATED).body(task);
    }
}
```

## 安全约定
- 密码使用 BCryptPasswordEncoder(12) 哈希
- 会话 cookie 配置：httpOnly、secure、sameSite
- 认证端点速率限制
- 错误消息不暴露内部细节
- CORS 限制为特定来源
- 使用 Spring Security Headers 配置安全头部

## CI/CD
- 每次 PR 自动运行：`mvn clean verify`
- 包含：lint、单元测试、集成测试、依赖审计
- 构建失败则阻止合并

## 性能预算
- API 响应时间：< 200ms（p95）
- 数据库查询：< 50ms（p95）
- JVM 堆内存：< 2GB（生产环境）
- GC 暂停：< 100ms

## Git 工作流
- 主干开发（trunk-based）
- 原子提交（~100 行/提交）
- 功能分支短寿命（< 2 天）
- 使用特性标志进行长期工作
- 提交消息格式：`feat: 添加任务创建端点`