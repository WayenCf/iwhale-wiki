---
type: source
tags: [代码分析, dms-repo-api, 存储抽象]
sources: []
created: 2026-04-20
updated: 2026-04-20
---


# 代码分析：dms-repo-api

> DMS 存储层抽象接口契约精读，所有 DAO 接口在此声明，具体实现由 dms-repo-mongodb 或 dms-repo-spool 提供。

## 模块职责

- **核心原则**：接口与实现分离，泛型化 CRUD，多存储引擎支持，多租户感知
- **文件数量**：36 个 Java 文件

### 包结构

```
com.iwhalecloud.ftf.dms.repo.api
├── Dao<T>               ← 泛型 CRUD 基础接口
├── AttachDao<T>         ← 文件附件抽象接口（[[wiki/dms90/实体/GridFS|GridFS]] 操作）
├── SearchIndex          ← 搜索索引接口（Elasticsearch）
├── TenantAware          ← 多租户感知接口
├── StorageType          ← 存储类型枚举（SPOOL/MONGODB）
├── DaoFactory           ← DAO 工厂（存储路由）
├── {Entity}Dao          ← 17 个业务实体 DAO 接口
└── util/UserContext      ← 用户上下文（安全认证+租户）
```

## 核心类

### Dao<T extends BaseEntity> — 泛型 CRUD 基础

| 方法 | 签名 | 说明 |
|------|------|------|
| create | `String create(T t)` | 创建实体，返回 ID |
| delete | `void delete(String id)` | 按 ID 删除 |
| update | `void update(T t)` | 更新实体（全量替换） |
| retrieve | `T retrieve(String id)` | 按 ID 查询 |

### AttachDao<T> — [[wiki/dms90/实体/GridFS|GridFS]] 文件接口

- uploadFile / downloadFile / deleteFile / renameFile / getMetadata / updateFile
- 多租户方法：uploadFileMultitenant / downloadFileMultitenant
- 默认方法：generateMetadata（Tika 全量提取）/ generateLightMetadata（轻量提取）/ extractContent（ES 索引用）

### DaoFactory — 存储路由

- 混合模式：storageType==null || 2 → MONGODB，否则 SPOOL
- 纯 SPOOL → SPOOL，纯 MONGO → MONGODB

## 关键发现

1. **接口分离 + 多实现**：Dao<T> 统一 CRUD 契约，AttachDao<T> 扩展 [[wiki/dms90/实体/GridFS|GridFS]] 文件操作
2. **ID 编码约定**：`{shardKey}-{realId}`（分隔符 `-`），多租户 `{userCode}-{shardKey}-{realId}`
3. **[[wiki/dms90/概念/多租户隔离|多租户感知]]**：TenantAware 接口优先 ThreadLocal，回退 Spring Security 上下文
4. **Tika 元数据提取**：AttachDao 默认方法提供全量和轻量两种模式，供 ES 索引用
5. **UserContext 双模式**：Portal 依赖模式解析 `userName@tenant`，独立模式解析 `userId@tenant@crt`

## 设计模式

- **接口分离原则**：Dao<T> / AttachDao<T> / 业务 DAO 三层接口
- **工厂模式**：DaoFactory + StorageType 实现运行时 DAO 路由
- **默认方法**：AttachDao 中 Java 8 default method 提供 Tika 元数据提取实现

## 关联概念

- [[wiki/dms90/概念/shardKey分表机制|shardKey 分表机制]]
- [[wiki/dms90/概念/多租户隔离|多租户隔离]]
- [[wiki/dms90/概念/双存储适配|双存储适配]]
- [[wiki/dms90/实体/GridFS|GridFS]]
- [[wiki/dms90/实体/MongoDB|MongoDB]]
