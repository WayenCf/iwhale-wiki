---
title: MongoDB
type: entity
created: 2026-04-19
updated: 2026-04-20
tags: [数据库, NoSQL, 文档存储, 现场问题, 代码分析]
sources: []
---

# MongoDB

> DMS 的默认存储后端，用于文档和文件的元数据存储与二进制文件管理（通过 GridFS）。

## 在 DMS 中的角色

- **默认后端**：通过 Spring Profile `mongodb` 激活（默认激活）
- **驱动版本**：MongoDB Java Driver 4.4.2
- **推荐版本**：MongoDB 4.4

## 存储模型

### 集合命名规则

| 类型 | 集合名 |
|------|--------|
| 文档元数据 | `Document-{shardKey}` |
| 文件元数据 | `File-{shardKey}` |
| 二进制内容 | GridFS（`fs.files` + `fs.chunks`） |

### 多租户隔离

| 模式 | 集合定位 |
|------|---------|
| `multiTenantEnabled=false`（默认） | 公共 DB，集合 `Document-{shardKey}` |
| `multiTenantEnabled=true` | 独立 DB `data_{userId}`，集合 `Document-{shardKey}` |

## 关键机制

- **shardKey 路由**：通过 `IdUtil.getShardKey()` 从复合 ID 中提取 shardKey，定位到对应集合
- **GridFS**：大文件分块存储，fileId 为 ObjectId
- **自动清理**：`documentService.autoDeleteDocInMongodb()` 定时任务

## 在现场问题中的角色

### 查询结果超限

- [[wiki/dms90/来源/2024-03-28 阿富汗下载PDF超出限制条数]]：单次查询超过 `ftf.dms.mongo.result-limit` 设定值（默认 10000）

### 连接超时

- [[wiki/dms90/来源/2024-04-23 DMS连接MongoDB超时]]：文件系统满导致服务不可用，默认超时 30000ms

### 数据库配置错误

- [[wiki/dms90/来源/2024-06-24 印尼项目DMS前台页面报500]]：项目自定义 `databasename` 和 `tenant_default` 导致连接错误的库（默认 admin）

## 关键配置

| 配置项 | 说明 | 默认值 |
|--------|------|--------|
| `ftf.dms.mongo.result-limit` | 单次查询返回最大条数 | 10000 |
| `spring.data.mongodb.uri` | 连接地址 | - |
| `spring.data.mongodb.database` | 数据库名 | admin |

## 常见故障模式

1. **文件系统满** → MongoDB 服务挂 → DMS 连接超时
2. **查询超限** → 业务报错 `exceeds the limit`
3. **数据库名配置错误** → 查询返回空 → 前台报错

## 代码级细节（来自代码分析）

### MongoRepository — 客户端管理

- 连接池配置：maxSize=50, minSize=10, maxConnectionIdleTime=1800s
- 认证：MongoCredential.createCredential(username, database, password)
- Profile：`@Profile("mongodb")` + `@Repository`
- 分片命令：`shardCollection db.coll key: {_id: "hashed"}`，覆盖 9 个核心集合

### 三级数据库隔离

| 模式 | 数据库名 | 说明 |
|------|---------|------|
| 单租户 | `tenant_default` 或 `tenantName`（配置值） | 所有租户共享 |
| 多租户（租户级） | `tenant_{tenantId}` | 每租户独立数据库 |
| 多租户（用户级） | `data_{userCode}` | 每用户独立数据库 |
| 全局 | `dms_database` | User、Token 等全局实体 |

### 集合命名规则（代码实现）

[[wiki/dms90/概念/shardKey分表机制|shardKey 分表]]的集合命名由 `MongoBase.getCollectionName()` 实现：

- 旧模式（`isEnabled=false`）：`Document-01`、`File-03`（横线分隔）
- 新模式（`isEnabled=true`）：`Document_01`、`File_03`（下划线分隔）
- shardKey 来源：AccessLog/TaskRecord 按月 `%02d`；Document/File/Material 由实体 getShardKey() 决定

### initTenantDatabase 初始化流程

创建租户时自动执行：
- Resource：初始化系统默认文件夹（从 CommonProperties.folders 读取）
- Release：creationDate 降序唯一索引
- AccessLog：creationDate(TTL 365天)、userId、result、consume、execution 索引
- Document：creationDate 降序索引
- Thumbnail：creationDate(TTL 365天) 索引
- TaskRecord：creationDate 降序索引
- DocumentLabel/FileLabel：entityId、labelType+labelContent、creationDate、updateDate 索引
- 如果是分片集群则执行 shardAllCollection()

### CGLIB 转换器体系

- Converter.toDBObject(obj)：Java 对象 → MongoDB DBObject（通过 CGLIB 代理）
- Converter.toObject(clazz, source)：DBObject → Java 对象
- 值传递逻辑：Date/Number/String/ObjectId/Boolean/Pattern/byte[]/UUID 直接传递，Map/Iterable/Array/Enum 创建代理

### 结果集限制

所有查询方法检查 `ret.size() >= mongoProperties.getResultLimit()`（**默认 10000**，可通过 `ftf.dms.mongo.result-limit` 配置修改），超出抛 DMSMongoException(EXCEED_RESULT_LIMIT)。

## 关联概念

- [[wiki/dms90/概念/shardKey分表机制|shardKey 分表机制]]
- [[wiki/dms90/概念/多租户隔离|多租户隔离]]
- [[wiki/dms90/概念/文档ID生命周期|文档 ID 生命周期]]
- [[wiki/dms90/概念/MongoDB连接超时]]
- [[wiki/dms90/概念/双存储适配|双存储适配]]
- [[wiki/dms90/概念/DMS MongoDB常用查询|MongoDB 常用查询]]

## 关联页面

- [[wiki/dms90/来源/2026-04-19 DMS项目概览|DMS 项目概览]]
- [[wiki/dms90/来源/2026-04-19 DocumentController接口逻辑梳理|DocumentController 接口]]
- [[wiki/dms90/来源/2026-04-19 FileController接口逻辑梳理|FileController 接口]]
- [[wiki/dms90/来源/2024-03-28 阿富汗下载PDF超出限制条数]]
- [[wiki/dms90/来源/2024-04-23 DMS连接MongoDB超时]]
- [[wiki/dms90/来源/2024-06-24 印尼项目DMS前台页面报500]]
- [[wiki/dms90/来源/2026-04-20 代码分析-dms-common|代码分析：dms-common]]
- [[wiki/dms90/来源/2026-04-20 代码分析-dms-repo-mongodb|代码分析：dms-repo-mongodb]]
- [[wiki/dms90/来源/2026-04-20 代码分析-dms-service-impl|代码分析：dms-service-impl]]
- [[wiki/dms90/来源/2026-04-20 代码分析-dms-repo-api|代码分析：dms-repo-api]]
- [[wiki/dms90/来源/2026-04-20 下载或查询报错文件不存在|下载或查询报错文件不存在]]
