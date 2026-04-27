---
title: GridFS
type: entity
created: 2026-04-20
updated: 2026-04-20
tags: [组件, MongoDB, 文件存储, 二进制存储]
sources: []
---

# GridFS

> MongoDB 的二进制大文件存储规范，DMS 使用 GridFS 存储文档/文件的实际内容（PDF、图片等），每个文件分块存储在 `.chunks` 集合中。

## 在 DMS 中的角色

| 方面 | 详情 |
|------|------|
| 用途 | 存储文档/文件/证书/资源的二进制内容 |
| 驱动 | MongoDB Java Driver 4.x GridFSBucket API |
| 文件 ID | ObjectId |

## 桶命名规则

桶名由实体类名（去掉 `Attach` 后缀）+ shardKey 拼接：

| 场景 | 桶名 | 底层集合 |
|------|------|---------|
| Document 无 shardKey | `Document` | `Document.files` + `Document.chunks` |
| Document 有 shardKey | `Document-{shardKey}` 或 `Document_{shardKey}` | `Document-{shardKey}.files` + `Document-{shardKey}.chunks` |
| 多租户 + shardKey | 同上，但在 `data_{userCode}` 数据库 | — |

分隔符由 `MongoNameProperties.isEnabled()` 控制：旧模式 `-`，新模式 `_`。

## 核心操作（MongoAttachDao<T>）

### 上传

1. generateLightMetadata() 设置轻量元数据（Content-Type + Filename）
2. prepareUploadStream() 预处理（DEPEND_ES 时 Tika 提取文本供 ES 索引）
3. 构建 GridFSUploadOptions，设置 metadata
4. 选择 GridFS 桶（按 shardKey 路由）
5. `gridFSBucket.uploadFromStream(filename, source, options)`

### 下载

- `gridFSBucket.openDownloadStream(new ObjectId(id))`
- 文件不存在时抛出 MaterialNotFoundException

### 更新（删旧存新）

GridFS 不支持原地更新，采用删除 + 重建策略，保持相同 ObjectId：

```java
BsonValue bv = new BsonObjectId(new ObjectId(id));
gridFSBucket.delete(bv);
gridFSBucket.uploadFromStream(bv, filename, source, options);
```

### 多租户操作

- `uploadFileMultitenant(userCode, tableSuffix, ...)` — 在 `data_{userCode}` 数据库操作
- `downloadFileMultitenant(userCode, tableSuffix, id)`
- `getMetadataMultiTenant(userCode, tableSuffix, id)`
- `updateFileMultiTenant(userCode, tableSuffix, ...)`
- `existMultitenant(userCode, tableSuffix, id)`

## AttachDao 接口层的默认方法

| 方法 | 说明 |
|------|------|
| generateMetadata() | Tika 全量元数据提取（Content-Type、文本内容等） |
| generateLightMetadata() | 轻量元数据（仅 Content-Type + Filename，不调 Tika） |
| extractContent() | 纯 Tika 内容提取 + reset()，供 ES 索引 |
| prepareUploadStream() | 上传流预处理（ByteArrayInputStream 直接提取，否则缓存临时文件） |

内部类 `UploadStreamContext`（AutoCloseable）封装上传流 + 提取内容 + 临时文件生命周期管理。

## 与 DMS 业务层的关系

- **DocumentServiceImpl**：uploadFile() → DocumentAttachDao.uploadFile()；downloadFile() → DocumentAttachDao.downloadFile()
- **CertificateServiceImpl**：importCrt() → CertificateAttachDao.uploadFile()
- **ResourceServiceImpl**：create() → ResourceAttachDao.uploadFile()
- **批量上传**：DocUploadTask → VolumeService.preCheck() → PdfUtil.sign() → DocumentAttachDao.uploadFile()

## 关联概念

- [[wiki/dms90/概念/shardKey分表机制|shardKey 分表机制]]
- [[wiki/dms90/概念/多租户隔离|多租户隔离]]
- [[wiki/dms90/概念/双存储适配|双存储适配]]
- [[wiki/dms90/概念/批量账单生成流程|批量账单生成流程]]

## 关联页面

- [[wiki/dms90/实体/MongoDB|MongoDB]]
- [[wiki/dms90/来源/2026-04-20 代码分析-dms-repo-mongodb|代码分析：dms-repo-mongodb]]
- [[wiki/dms90/来源/2026-04-20 代码分析-dms-service-impl|代码分析：dms-service-impl]]
