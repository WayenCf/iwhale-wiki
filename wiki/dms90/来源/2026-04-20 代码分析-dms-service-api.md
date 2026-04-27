---
type: source
tags: [代码分析, dms-service-api, 服务接口]
sources: []
created: 2026-04-20
updated: 2026-04-20
---


# 代码分析：dms-service-api

> DMS Service 层接口契约精读，所有 Service 接口在此声明，具体实现由 dms-service 模块提供。

## 模块职责

- **核心原则**：接口与实现分离，面向用例设计，统一入参出参，支持批量操作
- **文件数量**：24 个 Java 文件

### 包结构

```
com.iwhalecloud.ftf.dms.service.api
├── DocumentService       ← 文档管理（19 个方法）
├── FileService           ← 文件管理（与 DocumentService 对称）
├── FolderService         ← 文件夹管理（14 个方法，含递归复制）
├── ResourceService       ← 资源管理（11 个方法）
├── ReleaseService        ← 发布版本管理（11 个方法）
├── BatchDocumentService  ← 批量文档操作（5 个方法）
├── BatchFileService      ← 批量文件操作（与 BatchDocumentService 对称）
├── ExpImpService         ← 导入导出（4 个方法，异步任务模式）
├── IndexService          ← 搜索索引（8 个方法）
├── CertificateService    ← 证书管理
├── HistoryTemplateService ← 历史模板
└── 其他 13 个 Service 接口
```

## 核心类

### DocumentService — 文档管理

关键方法签名模式：
- `create(tenant, folderId, filename, source, map, type, userCode)` → 标准创建
- `createFromSync(tenant, ..., fileId, contentType)` → 同步创建（已有 fileId，跳过 [[wiki/dms90/实体/GridFS|GridFS]]）
- `delete(tenant, id, userCode)` → 级联删除（实体 + [[wiki/dms90/实体/GridFS|GridFS]] 文件 + 缩略图 + 标签 + 索引）
- `copy(tenant, id, targetFolderId, userCode)` → 复制（含 [[wiki/dms90/实体/GridFS|GridFS]] 文件副本）

### ReleaseService — 发布版本管理

- `create(tenant, release, releaseResources, withFile)` — withFile 控制是否复制附件
- `createWithHistoryDate()` — 含历史日期的发布
- `getCurrent()` / `getHistory()` — 版本查询

### ExpImpService — 异步导入导出

- `exportData/importData` → 返回任务 ID
- `getExportResult/getImportResult` → 轮询结果

## 关键发现

1. **方法签名统一模式**：几乎所有方法以 `tenant` 作为第一个参数，体现 [[wiki/dms90/概念/多租户隔离|多租户架构]] 贯穿性
2. **创建方法变体**：create() 标准创建 vs createFromSync() 同步创建（跳过 [[wiki/dms90/实体/GridFS|GridFS]] 上传）
3. **删除级联清理**：DocumentService.delete() 清理 5 个关联（实体 + 文件 + 缩略图 + 标签 + 索引）
4. **复制含存储层复制**：copy() 不仅复制实体，还复制 [[wiki/dms90/实体/GridFS|GridFS]] 文件，确保副本独立
5. **标签集合运算**：retrieveDocIdByLabels(labels, union) 支持并集/交集两种模式
6. **Sync 方法暗示外部系统**：createFromSync/updateContentFromSync 暗示 CMDB 等外部系统同步场景

## 设计模式

- **接口分离原则**：24 个 Service 接口各司其职
- **方法签名统一**：tenant 优先参数模式
- **创建变体**：create / createFromSync 双入口
- **异步任务模式**：ExpImpService 返回任务 ID + 轮询结果

## 关联概念

- [[wiki/dms90/概念/多租户隔离|多租户隔离]]
- [[wiki/dms90/概念/shardKey分表机制|shardKey 分表机制]]
- [[wiki/dms90/概念/双存储适配|双存储适配]]
- [[wiki/dms90/概念/批量账单生成流程|批量账单生成流程]]
- [[wiki/dms90/实体/GridFS|GridFS]]
