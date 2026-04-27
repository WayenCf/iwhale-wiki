---
title: Spring Boot
type: entity
created: 2026-04-20
updated: 2026-04-20
tags: [框架, Java, 应用框架]
sources: []
---

# Spring Boot

> DMS 全栈基于 Spring Boot 构建的应用框架，提供依赖注入、配置管理、安全认证、AOP、定时任务等核心能力。

## 在 DMS 中的角色

| 方面 | 详情 |
|------|------|
| 版本 | Spring Boot 2.x |
| 核心能力 | IoC/DI、自动配置、Spring Security、Spring MVC、Spring Data MongoDB |
| 部署模式 | independent（独立部署）/ integrated（嵌入 Portal） |

## 各模块的 Spring Boot 用法

### dms-common

- **配置类**：CommonConfiguration 加载 `classpath:config/common.properties`，注册 GuavaProperties / CommonProperties / CsvFieldProperties / MetricProperties
- **Bean 管理**：MetricConfiguration @PostConstruct 初始化 Sentinel 熔断规则
- **Profile**：`@Profile("mongodb")` 控制存储引擎激活

### dms-web

- **Spring Security**：SecurityConfiguration（independent 模式）和 PortalDependedSecurityConfiguration（integrated 模式）两套安全配置
- **WebMvcConfigurer**：WebConfiguration 注册拦截器链（流控 → 请求计数 → ThreadLocal 清理 → 文件类型校验）
- **@ControllerAdvice**：AppExceptionHandler 全局异常处理，映射 11 种异常到 HTTP 状态码
- **AOP**：LogAroundMethod @Aspect 拦截 API 访问日志
- **优雅停机**：GracefulShutdown 实现 TomcatConnectorCustomizer + ApplicationListener<ContextClosedEvent>
- **@ConditionalOnProperty**：大量条件化装配（CSRF、文件类型、流控、缓存刷新等）

### dms-service-impl

- **事务管理**：@Transactional 用于 DocumentServiceImpl / FileServiceImpl / FolderServiceImpl 等的 CUD 操作
- **@PostConstruct/@PreDestroy**：ReleaseSchedule 定时调度器生命周期管理
- **CommandLineRunner**：MqConfiguration / SentinelConfig 启动后初始化
- **线程池**：ServiceConfiguration 创建 4 个 Executor Bean

### dms-repo-mongodb

- **@Repository**：MongoRepository 使用 @Profile("mongodb") 激活
- **@Component**：MongoAlarmAspect AOP 告警切面

## 关键设计模式

- **双模式安全配置**：通过 `ftf.dms.mode` 配置切换 independent/integrated 两套 Spring Security 配置
- **条件化装配**：@ConditionalOnProperty / @ConditionalOnBean 大量使用，控制功能开关
- **Profile 分离**：@Profile("mongodb") / @Profile("spool") 实现存储引擎切换

## 关联页面

- [[wiki/dms90/来源/2026-04-20 代码分析-dms-common|代码分析：dms-common]]
- [[wiki/dms90/来源/2026-04-20 代码分析-dms-web|代码分析：dms-web]]
- [[wiki/dms90/来源/2026-04-20 代码分析-dms-service-impl|代码分析：dms-service-impl]]
- [[wiki/dms90/来源/2026-04-20 代码分析-dms-repo-mongodb|代码分析：dms-repo-mongodb]]
