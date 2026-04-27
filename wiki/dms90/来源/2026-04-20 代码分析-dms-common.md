---
type: source
tags: [代码分析, dms-common, 实体体系, 限流, 缓存, 安全]
created: 2026-04-20
updated: 2026-04-20
---

# 代码分析：dms-common


## 模块概述

- **职责**：为所有 DMS 子模块提供实体定义、配置体系、异常体系、流控机制、缓存基础设施、ID 生成/分片、安全工具、日志模块映射
- **根包**：`com.iwhalecloud.ftf.dms.common`
- **源文件数**：84 个主源文件 + 87 个测试文件
- **关键依赖**：Spring Boot、ZSmartCore、Sentinel、Guava Cache、FastJSON

## 核心发现

### 实体继承体系（30 个类）

```
BaseEntity → UserBaseEntity → AccessLog / Resource / File / Document / Folder / Release / TaskRecord / CertificateAndKey
BaseEntity → User / Tenant / Token / Thumbnail / Label
```

- **Resource**（DMS_RESOURCE）：核心资源实体，对应模版/子模版/数据适配器/图片
- **File**（DMS_FILE）→ **Document**（DMS_DOCUMENT）：文件与文档继承关系
- **Release** / **PreRelease**：发布版本管理
- **TaskRecord**（DMS_TASK_RECORD）：批量任务记录

### 双存储适配

通过 `Constant.DEPEND_*` 常量控制 MongoDB/Spool 双模式切换：
- **MongoDB 模式**：ID 格式 `{shardKey}-{realId}`
- **Spool 模式**：ID 为 base64 编码的相对路径
- **多租户模式**：ID 扩展为 `{userCode}-{shardKey}-{realId}`

### 异常体系

三分支异常层次：
- **BusinessException** — 业务异常（7 位错误码，含域/模块/序号）
- **SystemException** — 系统异常
- **MetricBlockException** — 限流熔断异常

### 限流机制

双层限流架构：
1. **自建流控**：按秒统计写入 CSV，`@Thread` 注解驱动后台线程
2. **Sentinel**：降级规则配置读写阈值

### 缓存架构

- **Guava 本地缓存**：进程内高速缓存
- **Redis 分布式刷新**：`RedisCacheRefreshTrigger` 每秒轮询 Redis Hash

### 安全加固

- **SecureUtil**：CRLF/XSS/路径穿越防护
- **XmlUtil/MimeMapping**：XXE 防护
- **FileUtil**：RSA 加密文件 ID

---

## 相关页面

- [[wiki/dms90/概念/shardKey分表机制]] — 分片路由机制
- [[wiki/dms90/概念/多租户隔离]] — 多租户 ID 与数据库隔离
- [[wiki/dms90/概念/限流机制]] — 双层限流架构
- [[wiki/dms90/概念/实体继承体系]] — 实体三层继承体系
- [[wiki/dms90/概念/双存储适配]] — MongoDB/Spool 双存储路由
- [[wiki/dms90/概念/异常体系]] — 三分支异常层次
- [[wiki/dms90/概念/缓存架构]] — Guava + Redis 缓存架构
- [[wiki/dms90/概念/安全加固]] — 安全工具与防护
