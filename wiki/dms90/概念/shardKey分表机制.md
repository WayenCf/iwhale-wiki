---
title: shardKey 分表机制
type: concept
created: 2026-04-19
updated: 2026-04-19
tags: [architecture, mongodb, sharding]
sources: []
---

# shardKey 分表机制

> DMS 通过 shardKey 实现 MongoDB 数据的水平分表，将文档/文件记录按 shardKey 路由到不同的集合中。

## 核心思想

避免单一 MongoDB 集合过大，通过 shardKey 将数据分散到 `Document-{shardKey}` / `File-{shardKey}` 等多个集合。

## shardKey 解析优先级

```
1. shardingKey（大写 K，接口参数）
2. shardingkey（小写 k，兼容参数）
3. 默认值：当前年 + 当前月（如 "20263"）
```

> **注意**：v2 批量接口 `/document/v2/make` 仅支持大写 K，无小写兼容。

## 复合 ID 构造

| 模式 | ID 格式 | 示例 |
|------|---------|------|
| 普通 | `{shardKey}-{docId}` | `20263-507f1f77bcf86cd799439011` |
| 多租户 | `{userId}-{shardKey}-{docId}` | `user123-20263-507f1f77bcf86cd799439011` |
| Spool | `base64(相对路径).replace("/","_")` | — |

## 集合命名

集合命名由 `MongoBase.getCollectionName()` 实现，分隔符可配置：

- **旧模式**（`MongoNameProperties.isEnabled=false`）：使用横线 `-`，如 `Document-01`、`File-03`
- **新模式**（`MongoNameProperties.isEnabled=true`）：使用下划线 `_`，如 `Document_01`、`File_03`

shardKey 来源：AccessLog/TaskRecord 按月 `%02d`；Document/File/Material 由实体 `getShardKey()` 决定。

## ID 解析（查询/下载时）

```
复合 ID 传入
  → IdUtil.getShardKey()  提取 shardKey → 定位集合
  → IdUtil.getRealId()    提取 docId   → 查询 _id
  → 从记录取 fileId（GridFS ID）       → 下载二进制
```

## 适用场景

- 文档生成、上传、下载、删除全流程贯穿
- 批量操作中每个文档可独立指定 shardKey
- 按 shardKey 删除整个分表的数据

## 关联页面

- [[wiki/dms90/实体/MongoDB|MongoDB]]
- [[wiki/dms90/概念/多租户隔离|多租户隔离]]
- [[wiki/dms90/概念/文档ID生命周期|文档 ID 生命周期]]
- [[wiki/dms90/来源/2026-04-19 DocumentController接口逻辑梳理|DocumentController 接口]]
- [[wiki/dms90/来源/2026-04-19 FileController接口逻辑梳理|FileController 接口]]
