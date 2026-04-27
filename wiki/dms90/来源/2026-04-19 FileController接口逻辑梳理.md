---
title: FileController 接口逻辑梳理
type: source
created: 2026-04-19
updated: 2026-04-19
tags: [api, file, controller, java]
sources: []
---


# FileController 接口逻辑梳理

> DMS 文件管理控制器，处理文件的 upload/download、限流、容量预检、多租户隔离。路径前缀 `/file`。

## 元数据

| 字段 | 值 |
|------|-----|
| 来源 | FileController接口逻辑梳理.md |
| 源文件 | `dms/dms-web/.../api/FileController.java` |
| 路径前缀 | `/file` |

## 接口总览

| 分类 | 接口 | 方法 | 核心功能 |
|------|------|------|---------|
| 上传 | `POST /file` | upload | multipart 文件上传 |
| 下载 | `GET /file/{id}` | download | 单文件下载 / 获取 token URL |
| 下载 | `POST /file/batch` | batch | 批量下载（ZIP） |
| 下载 | `POST /file/labels` | labels | 按标签下载 |
| 查询 | `GET /file/metadata/{id}` | metadata | 单文件元数据 |
| 查询 | `POST /file/metadata` | metadata batch | 批量元数据 |
| 操作 | `PATCH /file/{id}` | rename | 重命名 |
| 查询 | `POST /file/query` | query | 按标签+ID 查询 |
| 删除 | `DELETE /file/{id}` | delete | 删除文件+二进制+索引 |
| 删除 | `POST /file/shardKey/delete` | delete by shard | 按分表键删除 |
| 删除 | `POST /file/storePath/delete` | delete by path | 按存储路径删除（Spool） |
| 缩略图 | `GET /file/{id}/{w}/{h}` | thumbnail | 图片缩略图 |

## 关键要点

### 上传流程

1. Content-Length 欺骗检测（可选）
2. 上传限流 `executeUpload()`
3. 安全校验 → 空文件校验 → 图片翻转处理（可选）
4. 父目录租户鉴权
5. 容量预检 `volumeService.preCheck()`
6. 构建 File 实体 → `FileUploadTask.call()` → GridFS 存储
7. 标签绑定 → 返回文件 ID

### 文件 ID 构造规则

| 模式 | 格式 |
|------|------|
| Spool | `base64(相对路径).replace("/","_")`（可加密） |
| MongoDB 普通 | `{shardKey}-{aid}` |
| MongoDB 多租户 | `{userId}-{shardKey}-{aid}` |

### shardKey 解析

优先级：`shardingKey`（大写K）> `shardingkey`（小写k）> 默认年月（如 `20263`）

### 中国电信 DFS 迁移模式

`ftf.dms.china.telecom.mode=true` + 超过 `dfsTime` 时，从 DFS 路径读取存量文件。

### 容量控制

- 上传前：`volumeService.preCheck()` 预检
- 失败时：`volumeService.rollback()` 回退计数

## 提到的实体

- [[wiki/dms90/实体/MongoDB|MongoDB]]

## 涉及的概念

- [[wiki/dms90/概念/shardKey分表机制|shardKey 分表机制]]
- [[wiki/dms90/概念/多租户隔离|多租户隔离]]
- [[wiki/dms90/概念/限流机制|限流机制（Sentinel）]]
- [[wiki/dms90/概念/文档ID生命周期|文档 ID 生命周期]]
