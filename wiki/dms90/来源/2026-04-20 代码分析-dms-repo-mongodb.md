---
type: source
tags: [代码分析, dms-repo-mongodb, MongoDB实现]
sources: []
created: 2026-04-20
updated: 2026-04-20
---


# 代码分析：dms-repo-mongodb

> DMS 存储层 MongoDB 实现精读，负责 MongoDB 连接管理、集合路由（shardKey 分表）、[[wiki/dms90/实体/GridFS|GridFS]] 文件读写、多租户数据库隔离、对象-文档转换（CGLIB 代理）、[[wiki/dms90/实体/Sentinel|Sentinel]] 流控熔断、告警监控。

## 模块职责

- **文件数量**：40 个 Java 文件
- **关键依赖**：MongoDB Java Driver 4.x、[[wiki/dms90/实体/Sentinel|Sentinel]]、CGLIB、Tika

### 包结构

```
com.iwhalecloud.ftf.dms.repo.mongodb
├── MongoRepository      ← MongoDB 客户端+连接池+分片管理
├── MongoBase<T>         ← 抽象基类（集合路由+数据库选择+[[wiki/dms90/实体/GridFS|GridFS]] 桶选择）
├── MongoDao<T>          ← 抽象 DAO（CRUD 实现+Sentinel 流控）
├── MongoAttachDao<T>    ← 抽象附件 DAO（[[wiki/dms90/实体/GridFS|GridFS]] 读写）
├── Mongo{Entity}Dao     ← 17 个具体业务 DAO 实现
├── converter/           ← CGLIB 代理对象↔DBObject 双向转换
├── alarm/               ← AOP 告警切面
└── config/              ← MongoDB 连接配置属性
```

## 核心类

### MongoRepository — 客户端管理

- 连接池配置：maxSize=50, minSize=10, maxConnectionIdleTime=1800s
- 认证：MongoCredential.createCredential
- 分片：shardCollection 执行哈希分片 `{_id: "hashed"}`，覆盖 9 个核心集合

### MongoBase<T> — 集合路由核心

- **数据库选择**：非多租户 → `tenant_default`；多租户（租户级）→ `tenant_{tenantId}`；多租户（用户级）→ `data_{userCode}`
- **[[wiki/dms90/概念/shardKey分表机制|集合路由]]**：`{EntityName}-{shardKey}`（旧模式）或 `{EntityName}_{shardKey}`（新模式），由 `MongoNameProperties.isEnabled()` 控制分隔符
- **[[wiki/dms90/实体/GridFS|GridFS]] 桶选择**：去 Attach 后缀 + shardKey 拼接桶名

### MongoDao<T> — CRUD 实现

- create：自动生成 ObjectId + [[wiki/dms90/实体/Sentinel|Sentinel]] 写流控 + 4 种异常映射
- retrieve：[[wiki/dms90/实体/Sentinel|Sentinel]] 读流控 + 结果集限制（默认 10000）
- buildFilters：反射构建等值过滤条件

### MongoAttachDao<T> — GridFS 读写

- uploadFile：轻量元数据 → Tika 提取（ES 索引）→ GridFS 桶上传
- updateFile：「删旧存新」策略，保持相同 ObjectId
- 多租户：在 `data_{userCode}` 数据库上操作

## 关键发现

1. **[[wiki/dms90/概念/shardKey分表机制|shardKey 路由]]**：写入时实体 getShardKey() 决定集合；读取时 IdUtil.getKeyAndId() 解析 shardKey；批量查询按 shardKey 分组
2. **[[wiki/dms90/概念/多租户隔离|三级数据库隔离]]**：`tenant_default`（共享）→ `tenant_{tenantId}`（租户级）→ `data_{userCode}`（用户级），全局实体始终在 `dms_database`
3. **[[wiki/dms90/实体/GridFS|GridFS 更新策略]]**：不支持原地更新，采用删除+重建，保持 ObjectId
4. **[[wiki/dms90/实体/Sentinel|Sentinel]] 流控**：所有读写操作包裹在 Entry 中，降级→StorageDegradeException，阻断→MetricBlockException
5. **CGLIB 转换器**：Converter.toDBObject/toObject 双向转换，支持 Map/Iterable/Array/Enum 代理

## 数据库结构

### 关键索引

| 集合 | 索引 | 类型 |
|------|------|------|
| Release | creationDate: -1 | 唯一 |
| AccessLog | creationDate: -1 | TTL 365天 |
| Document | creationDate: -1 | 普通 |
| DocumentLabel | labelType:1, labelContent:1 | 普通 |

### initTenantDatabase 初始化

- Resource：系统默认文件夹
- Release/AccessLog/Document/Thumbnail/TaskRecord/DocumentLabel/FileLabel 各创建索引

## 设计模式

- **模板方法**：MongoBase 提供集合路由，MongoDao/MongoAttachDao 实现具体操作
- **代理模式**：CGLIB 代理实现 Java 对象与 DBObject 双向透明转换
- **AOP 告警**：MongoAlarmAspect 切面拦截所有 MongoDB 操作

## 关联概念

- [[wiki/dms90/概念/shardKey分表机制|shardKey 分表机制]]
- [[wiki/dms90/概念/多租户隔离|多租户隔离]]
- [[wiki/dms90/概念/双存储适配|双存储适配]]
- [[wiki/dms90/实体/GridFS|GridFS]]
- [[wiki/dms90/实体/MongoDB|MongoDB]]
- [[wiki/dms90/实体/Sentinel|Sentinel]]
