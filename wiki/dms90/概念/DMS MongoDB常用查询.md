---
type: concept
tags: [MongoDB, 查询, 运维, 问题排查]
sources: []
created: 2026-04-20
updated: 2026-04-20
---

# DMS MongoDB 常用查询语句

> DMS 系统 MongoDB 数据库的常用查询语句汇总，用于现场问题排查和运维操作。

---

## 基础查询

### ⭐ 复合 ID 查询（重要）

DMS 返回给客户端的 ID 格式为 **复合 ID**：`{shardKey}-{ObjectId}`

```
示例：20263-69c34a988d7ebc1b298e70ac
        │     │
        │     └─ ObjectId（MongoDB 的 _id 字段值）
        └─ shardKey（年月，用于确定集合名）
```

**正确查询方式**：

```javascript
// 客户端返回的复合 ID：20263-69c34a988d7ebc1b298e70ac

// 步骤 1：解析出 shardKey 和 ObjectId
shardKey = "20263"           // 集合名：File-20263 或 Document-20263
objectId = "69c34a988d7ebc1b298e70ac"  // MongoDB 的 _id

// 步骤 2：查询（只用 ObjectId，不用完整复合 ID）
db.getCollection("File-20263").findOne({ "_id": "69c34a988d7ebc1b298e70ac" });

// ❌ 错误：不要用完整复合 ID 查询 _id
db.getCollection("File-20263").findOne({ "_id": "20263-69c34a988d7ebc1b298e70ac" }); // 查不到
```

**多租户复合 ID**：`{userId}-{shardKey}-{ObjectId}`

```
示例：user123-20263-69c34a988d7ebc1b298e70ac
        │       │     │
        │       │     └─ ObjectId
        │       └─ shardKey
        └─ userId

查询时同样只用 ObjectId 部分：
db.getCollection("File-20263").findOne({ "_id": "69c34a988d7ebc1b298e70ac" });
```

详见：[[wiki/dms90/概念/文档ID生命周期]]、[[wiki/dms90/概念/shardKey分表机制]]

---

### 查询最新的文档记录

```javascript
use tenant_default;

db.getCollection("Document").find().sort({ "creationDate": -1 });
```

**说明**：
- 查询所有 Document 记录，按创建时间降序排列
- Document 是文档元数据表，不分表（或使用默认分表）
- 用于查看最新创建/上传的文档

### 查询最新的资源记录

```javascript
use tenant_default;

db.getCollection("Resource").find().sort({ "creationDate": -1 });
```

**说明**：
- 查询所有 Resource 记录，按创建时间降序排列
- Resource 是模版/子模板/数据适配器等资源的元数据表
- 用于查看最新上传/发布的模版
- 可用于排查"模版找不到"问题（[[wiki/dms90/概念/模版导入]]）

### 查询指定月份的文件记录

```javascript
use tenant_default;

db.getCollection("File-20263").find().sort({ "creationDate": -1 });
```

**说明**：
- 查询 shardKey=20263（2026年3月）的文件记录
- File 集合按 shardKey 分表：`File-{YYYYMM}`
- 用于查看特定月份的上传文件
- shardKey 构造规则详见 [[wiki/dms90/概念/shardKey分表机制]]

---

## 按需查询示例

### 查询指定模版

```javascript
// 查询 eRFpage.jasper 模版
db.getCollection("Resource").find({ "fileName": "eRFpage.jasper" }).pretty();

// 模糊查询
db.getCollection("Resource").find({ "fileName": { $regex: /eRFpage/i } }).pretty();

// 查询所有 .jasper 模版
db.getCollection("Resource").find({ "fileName": /\.jasper$/ }, { fileName: 1, customPath: 1, _id: 0 }).limit(10);
```

### 查询指定文件

```javascript
// 按 ID 查询（重要：复合 ID 的正确用法）
// 如果客户端返回的 ID 是：20263-69c34a988d7ebc1b298e70ac
// 其中：
//   - 20263 是 shardKey（用于确定集合名：File-20263）
//   - 69c34a988d7ebc1b298e70ac 是 MongoDB ObjectId（实际的 _id 字段值）

// 正确查询方式：只使用 ObjectId 部分
db.getCollection("File-20263").findOne({ "_id": "69c34a988d7ebc1b298e70ac" });

// 错误查询方式：不要用完整的复合 ID 查询 _id 字段
// db.getCollection("File-20263").findOne({ "_id": "20263-69c34a988d7ebc1b298e70ac" }); // ❌ 查不到

// 按文件名查询
db.getCollection("File-20263").find({ "fileName": { $regex: /文件名/i } }).pretty();
```

**复合 ID 解析说明**：

```
客户端接收：20263-69c34a988d7ebc1b298e70ac
                │     │
                │     └─ ObjectId 部分 → MongoDB 的 _id 字段
                └─ shardKey 部分 → 用于确定集合名 File-20263

查询时：
  1. 从 ID 中提取 shardKey = "20263" → 集合名 = "File-20263"
  2. 从 ID 中提取 ObjectId = "69c34a988d7ebc1b298e70ac" → 查询条件 _id
  3. 执行查询：db.getCollection("File-20263").findOne({ "_id": "69c34a988d7ebc1b298e70ac" })
```

详见 [[wiki/dms90/概念/文档ID生命周期]] 和 [[wiki/dms90/概念/shardKey分表机制]]。

### 查询任务记录

```javascript
// 查询最新的任务记录
db.getCollection("TaskRecord-20263").find().sort({ "creationDate": -1 }).limit(10);

// 查询运行中的任务
db.getCollection("TaskRecord-20263").find({ "status": "running" }).pretty();

// 查询失败的任务
db.getCollection("TaskRecord-20263").find({ "status": "failed" }).sort({ "creationDate": -1 }).limit(10);
```

---

## 分表相关查询

### 列出所有 DMS 集合

```javascript
// 列出所有集合名称
db.getCollectionNames().forEach(function(name) {
  if (name.indexOf('DMS_') >= 0 || name.indexOf('Document') >= 0 || name.indexOf('File-') >= 0) {
    print(name);
  }
});

// 统计各类型集合数量
db.getCollectionNames().filter(function(name) {
  return name.indexOf('Document') >= 0;
}).length;

db.getCollectionNames().filter(function(name) {
  return name.indexOf('File-') >= 0;
}).length;
```

### 跨分表查询

```javascript
// 查询所有月份的 File 集合
db.getCollectionNames().forEach(function(name) {
  if (name.indexOf('File-') >= 0) {
    var count = db.getCollection(name).count();
    if (count > 0) {
      print(name + ': ' + count + ' records');
    }
  }
});
```

---

## 多租户相关查询

### 切换到不同数据库

```javascript
// 默认数据库
use tenant_default;

// 租户级独立数据库
use tenant_<tenantId>;

// 用户级独立数据库（多租户模式）
use data_<userCode>;
```

**说明**：详见 [[wiki/dms90/概念/多租户隔离]] 的三级数据库隔离。

### 跨数据库查询

```javascript
// 查询其他数据库的集合
db.getSiblingDB('data_COC_2604').getCollection("Resource").findOne({ "fileName": "eRFpage.jasper" });

// 查询全局数据库（User、Token 等）
db.getSiblingDB('dms_database').getCollection("User").findOne({ "userCode": "admin" });
```

---

## GridFS 相关查询

### 查询 GridFS 文件

```javascript
// 查询 GridFS 元数据
db.getCollection("fs.files").find().sort({ uploadDate: -1 }).limit(10);

// 按 fileId 查询
db.getCollection("fs.files").findOne({ "filename": "文件ID" });

// 统计 GridFS 文件总数
db.getCollection("fs.files").count();
```

**说明**：GridFS 用于存储大文件，详见 [[wiki/dms90/实体/GridFS]]。

---

## 统计与聚合查询

### 统计文档数量

```javascript
// 统计总文档数
db.getCollection("Document").count();

// 按日期统计文档数
db.getCollection("Document").aggregate([
  { $group: { _id: { $dateToString: { format: "%Y-%m-%d", date: "$creationDate" } }, count: { $sum: 1 } } },
  { $sort: { _id: -1 } }
]);

// 按用户统计文档数
db.getCollection("Document").aggregate([
  { $group: { _id: "$userId", count: { $sum: 1 } } },
  { $sort: { count: -1 } }
]);
```

### 统计文件大小

```javascript
// 统计总存储大小（字节）
db.getCollection("File-20263").aggregate([
  { $group: { _id: null, totalSize: { $sum: "$fileSize" } } }
]);

// 统计每个文件类型的大小
db.getCollection("File-20263").aggregate([
  { $group: { _id: "$contentType", totalSize: { $sum: "$fileSize" }, count: { $sum: 1 } } },
  { $sort: { totalSize: -1 } }
]);
```

---

## 性能优化查询

### 查看索引

```javascript
// 查看集合的所有索引
db.getCollection("Document").getIndexes();

// 查看索引大小
db.getCollection("Document").getIndexStats();
```

### 查看集合统计

```javascript
// 查看集合统计信息
db.getCollection("Document").stats();

// 查看集合大小（字节）
db.getCollection("Document").dataSize();
db.getCollection("Document").storageSize();
db.getCollection("Document").totalSize();
```

---

## 故障排查查询

### 查询异常文档

```javascript
// 查询没有关联文件的文档
db.getCollection("Document").find({ "fileId": { $exists: false } }).limit(10);

// 查询 fileCount 为 0 的文档
db.getCollection("Document").find({ "fileCount": 0 }).limit(10);
```

### 查询孤儿文件

```javascript
// 查询没有被任何文档引用的文件
db.getCollection("File-20263").find({ "referenceCount": 0 }).limit(10);
```

---

## 常用查询组合

### 组合 1：排查模版问题

```javascript
// 1. 查询模版是否存在
use tenant_default;
db.getCollection("Resource").findOne({ "fileName": "eRFpage.jasper" });

// 2. 如果不存在，查询所有 jasper 模版看路径格式
db.getCollection("Resource").find({ "fileName": /\.jasper$/ }, { fileName: 1, customPath: 1 }).limit(5);

// 3. 查询模版的发布记录
db.getCollection("DMS_RELEASE_RESOURCE").find({ "fileName": "eRFpage.jasper" }).pretty();
```

### 组合 2：排查上传问题

```javascript
// 1. 查询最新的上传记录
db.getCollection("File-20263").find().sort({ "creationDate": -1 }).limit(10);

// 2. 查询对应的文档记录
db.getCollection("Document").find({ "fileId": "从上一步获取的fileId" });

// 3. 查询对应的任务记录
db.getCollection("TaskRecord-20263").find({ "fileId": "从上一步获取的fileId" }).sort({ "creationDate": -1 }).limit(5);
```

### 组合 3：排查性能问题

```javascript
// 1. 查看集合大小
db.getCollection("Document").stats();

// 2. 查看索引使用情况
db.getCollection("Document").getIndexes();

// 3. 查询慢查询（需启用慢查询日志）
// 查看 MongoDB 日志文件或使用 database profiler
```

---

## 相关页面

- [[wiki/dms90/实体/MongoDB]] — MongoDB 存储后端详解
- [[wiki/dms90/概念/shardKey分表机制]] — 分表路由机制
- [[wiki/dms90/概念/文档ID生命周期]] — 复合 ID 生成与解析
- [[wiki/dms90/概念/多租户隔离]] — 多租户数据库隔离
- [[wiki/dms90/实体/GridFS]] — GridFS 二进制存储
- [[wiki/dms90/概念/模版导入]] — 模版导入流程
- [[wiki/dms90/来源/2026-04-20 代码分析-dms-repo-mongodb]] — MongoDB 实现代码分析

---

*最后更新：2026-04-22*
*来源：CongoB 项目现场排查经验*
