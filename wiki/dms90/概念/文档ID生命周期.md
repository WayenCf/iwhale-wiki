---
title: 文档 ID 生命周期
type: concept
created: 2026-04-19
updated: 2026-04-19
tags: [architecture, id, lifecycle]
sources: []
---

# 文档 ID 生命周期

> DMS 中文档/文件的 ID 从生成到使用的完整流程，涉及 shardKey 路由、复合 ID 构造和多租户前缀。

## 生成阶段

```
1. rawId = idGenerator.get()          → MongoDB ObjectId，存入 _id
2. shardKey = 参数值 or 默认年月       → 存入 shardKey 字段
3. aid = GridFS 存储 ObjectId         → 存入 fileId 字段
4. 返回给客户端的复合 ID：
     普通：{shardKey}-{aid}
     多租户：{userId}-{shardKey}-{aid}
```

## 查询/下载阶段

```
客户端传入复合 ID
  → IdUtil.getShardKey()  → 提取 shardKey → 定位集合 Document-{shardKey}
  → IdUtil.getRealId()    → 提取 rawId    → 查询 _id = rawId
  → 从记录取 fileId（GridFS ID）
  → 下载二进制文件流
```

## Spool 模式差异

- 返回 ID = `base64(相对路径).replace("/","_")`（可选加密）
- 下载时长度 > 60 触发 Spool ID 预处理：`_` 还原为 `/` → 解密（可选）

## Document 与 File 的 ID 格式对比

| 维度 | Document ID | File ID |
|------|------------|---------|
| 集合前缀 | `Document-{shardKey}` | `File-{shardKey}` |
| 响应头 | `Document-Id`（GridFS fileId） | 无 |
| 响应 body | 复合 ID | 复合 ID |
| Spool 模式 | `doc.getFileId().replace("/","_")` | `base64(相对路径).replace("/","_")` |

## 注意事项

- 响应头 `Document-Id` 返回的是 GridFS fileId，响应 body 返回的是复合 ID — 下载时应使用 **body 中的复合 ID**
- v2 批量接口的 shardingKey 仅支持大写 K
- shardKey 默认值 = 年月字符串（如 `20263`），必然非空

## 关联页面

- [[wiki/dms90/概念/shardKey分表机制|shardKey 分表机制]]
- [[wiki/dms90/概念/多租户隔离|多租户隔离]]
- [[wiki/dms90/来源/2026-04-19 DocumentController接口逻辑梳理|DocumentController 接口]]
- [[wiki/dms90/来源/2026-04-19 FileController接口逻辑梳理|FileController 接口]]
