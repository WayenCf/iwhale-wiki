---
title: Guava Cache
type: entity
created: 2026-04-20
updated: 2026-04-20
tags: [组件, 缓存, 本地缓存]
sources: []
---

# Guava Cache

> Google Guava 提供的本地内存缓存，DMS 使用 Guava Cache 作为一级缓存，配合 Redis 分布式刷新触发器实现集群缓存一致性。

## 在 DMS 中的角色

| 方面 | 详情 |
|------|------|
| 来源 | Google Guava |
| Maven 坐标 | com.google.guava:guava |
| 用途 | 本地内存缓存，减轻 MongoDB 查询压力 |
| 集成模块 | dms-common（LocalCache 实现）、dms-service-impl（业务缓存） |

## LocalCache 实现（dms-common/cache 包）

### 缓存构建

```java
CacheBuilder.newBuilder()
    .initialCapacity(initialCapacity)
    .maximumSize(maximumSize)
    .expireAfterAccess(expireTime, timeUnit)
    .concurrencyLevel(concurrencyLevel)
    .build();
```

### 配置属性（GuavaProperties，前缀 `ftf.dms.guava`）

| 属性 | 类型 | 用途 |
|------|------|------|
| initialCapacity | Integer | 初始容量 |
| maximumSize | Integer | 最大容量 |
| expireTime | Integer | 过期时间 |
| timeUnit | TimeUnit | 时间单位 |
| concurrencyLevel | Integer | 并发级别 |

### 操作方法

`put()`, `get()`, `remove()`, `clear()`, `size()`, `getAll()`

### CacheKey<T> — 复合缓存键

| 字段 | 类型 | 含义 |
|------|------|------|
| tenant | String | 租户 |
| type | Class<T> | 缓存数据类型 |
| version | String | 版本 |
| key | String | 业务键 |

重写 equals/hashCode 基于 4 字段，toString() 输出 JSON。

## Redis 分布式刷新触发器

### RedisCacheRefreshTrigger

- 条件：`ftf.dms.cache.refresh.enabled=true`
- 实现 CommandLineRunner，启动后每秒轮询 Redis Hash
- Redis Hash key 结构：`refreshCache` → { expectedRefreshTime, {nodeId}_lastRefreshTime, {nodeId}_lastCheckTime }
- 当 `expectedRefreshTime > lastRefreshTime` 时，执行 LocalCache.clear()
- 超过 6 秒未更新 lastCheckTime 的节点视为已停止，清除其 key
- nodeId 取自环境变量 CLOUD_NODE_ID 或主机名

## 业务层使用（dms-service-impl）

### ReleaseServiceImpl — 双重检查锁缓存

```java
synchronized(lock) {
    Release release = LocalCache.get(cacheKey);
    if (release == null) {
        release = releaseDao.retrieve(id);
        LocalCache.put(cacheKey, release);
    }
}
```

### UserServiceImpl — 用户缓存

- `retrieve(id)` 使用 LocalCache 缓存用户对象

### DocEncryptor — KeyStore 缓存

- `getKeyStore(cak)` 使用 LocalCache 缓存已加载的 PKCS12 KeyStore

### DocMakeTaskFactory — 切割规则缓存

- 切割规则（JSON → Rule → SegmentDef）缓存到 LocalCache

### BatchDocMakeTask — 模板缓存

- `TEMPLATE_CACHE` 缓存已编译的 JasperReports 模板

## 关联概念

- [[wiki/dms90/概念/缓存架构|缓存架构]]

## 关联页面

- [[wiki/dms90/来源/2026-04-20 代码分析-dms-common|代码分析：dms-common]]
- [[wiki/dms90/来源/2026-04-20 代码分析-dms-service-impl|代码分析：dms-service-impl]]
- [[wiki/dms90/实体/Redis|Redis]]
