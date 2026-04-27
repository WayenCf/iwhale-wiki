---
title: DocumentController 接口逻辑梳理
type: source
created: 2026-04-19
updated: 2026-04-19
tags: [api, document, controller, java]
sources: []
---


# DocumentController 接口逻辑梳理

> DMS 核心控制器，处理文档的全生命周期：生成、上传、下载、更新、删除、查询。路径前缀 `/document`。

## 元数据

| 字段 | 值 |
|------|-----|
| 来源 | DocumentController接口逻辑梳理.md |
| 源文件 | `dms/dms-web/.../api/DocumentController.java` |
| 路径前缀 | `/document` |

## 接口总览（23 个端点）

| 分类 | 接口 | 方法 | 核心功能 |
|------|------|------|---------|
| 生成 | `/document/make` | POST | 单文档生成（模板 + 数据） |
| 生成 | `/document/v2/make` | POST | 批量 JSON 请求体生成 |
| 上传 | `/document/upload` | POST | 本地文件路径上传 |
| 下载 | `/document/{id}` | GET | 单文档下载/预览/获取 token |
| 下载 | `/document` | POST | 批量下载（ZIP） |
| 下载 | `/document/single` | GET | 批量生成文档的单个下载 |
| 下载 | `/document/batch` | POST | 批量生成文档的批量下载 |
| 下载 | `/document/labels` | POST | 按标签下载（支持 PDF/CSV/TXT 合并） |
| 下载 | `/document/{id}/**` | GET | 重定向到模板关联资源 |
| 更新 | `/document/update/{id}` | POST | 替换文档内容 |
| 删除 | `/document/{id}` | DELETE | 删除文档及二进制 |
| 重命名 | `/document/{id}` | PATCH | 修改文档名称 |
| 查询 | `/document/metadata/{id}` | GET | 单文档元数据 |
| 查询 | `/document/metadata` | POST | 批量元数据 |
| 查询 | `/document/query` | POST | 按标签/ID 查询 |
| 其他 | `/document/demo` | GET/POST | 获取模板示例 XML |
| 其他 | `/document/sigPosition/{id}` | GET | PDF 签名坐标 |
| 其他 | `/document/recent` | GET | 最近文档列表 |
| 监控 | `/document/monitor/{pageNum}` | GET | 离线任务进度 |
| 监控 | `/document/monitor/count` | GET | 离线任务数量 |
| 删除 | `/document/shardKey/delete` | POST | 按分表键删除 |
| 删除 | `/document/storePath/delete` | POST | 按存储路径删除（Spool） |

## 关键要点

### 文档生成核心流程（`/make`）

1. 封装参数 → 限流 → 安全校验 → 模板鉴权
2. 解析模板版本（支持按历史时间点回溯）
3. 生成文档 ID（shardKey-docId 格式）
4. 同步/异步分支：`DocMakeTask` vs `HeavyDocMakeTask`
5. 持久化 → 绑定标签

### 文档 ID 机制

```
普通模式:    {shardKey}-{docId}
多租户模式:  {userId}-{shardKey}-{docId}
```

shardKey 默认 = 当前年月（如 `20263`），用于 MongoDB 集合分表路由。

### 同步 vs 异步

| | 同步 | 异步 |
|---|---|---|
| 触发 | 默认 | `async=true` 或传入 `xmlPath` |
| 执行 | 当前线程阻塞 | 线程池 + MQ 通知 |
| 结果 | 响应头 Document-Id | MQ 推送 |

### 按标签下载的合并格式

- `pdf` → PDFBox 合并
- `csv` → 跳过后续表头行
- `txt` → 逐行合并
- 空 → ZIP 压缩

### 重新生成（regenerate）

下载时文件丢失 + `regenerate=true` → 从 GridFS 取原始 XML 数据 → 重新渲染 → 覆盖存储

## 提到的实体

- [[wiki/dms90/实体/MongoDB|MongoDB]]
- [[wiki/dms90/实体/JasperReports|JasperReports]]

## 涉及的概念

- [[wiki/dms90/概念/shardKey分表机制|shardKey 分表机制]]
- [[wiki/dms90/概念/多租户隔离|多租户隔离]]
- [[wiki/dms90/概念/限流机制|限流机制（Sentinel）]]
- [[wiki/dms90/概念/文档ID生命周期|文档 ID 生命周期]]
