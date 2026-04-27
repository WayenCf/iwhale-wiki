---
type: concept
tags: [故障, MongoDB, 连接超时, 数据库]
sources:
  - 20240423-DMS连接MongDB超时Timed out after 30000 ms
  - 20240624-印尼项目DMS前台页面打开报500
created: 2024-04-23
updated: 2026-04-20
---

# MongoDB连接超时

> DMS 连接 MongoDB 超时问题，通常由服务不可用或配置错误导致。

## 在现场问题中的角色

### 文件系统满导致服务挂

- [[wiki/dms90/来源/2024-04-23 DMS连接MongoDB超时]]：MongoDB 文件系统满，服务挂掉，DMS 连接超时 30000ms

### 数据库配置错误

- [[wiki/dms90/来源/2024-06-24 印尼项目DMS前台页面报500]]：项目自定义 databasename 导致连接错误的库，查询返回空

## 排查步骤

1. 检查 MongoDB 配置连接是否正确
2. `telnet` MongoDB 连接地址是否通
3. 检查 MongoDB 服务运行状态
4. 检查文件系统空间
5. 检查数据库名称配置

## 常见原因

| 原因 | 表现 | 解决 |
|------|------|------|
| 文件系统满 | 服务不可用 | 清理空间，重启 MongoDB |
| 配置错误 | 查询返回空 | 修正 databasename/tenant_default |
| 网络不通 | telnet 失败 | 检查网络和防火墙 |
| 服务未启动 | 连接拒绝 | 启动 MongoDB 服务 |

## 相关实体

- [[wiki/dms90/来源/2026-04-19 DMS项目概览|DMS]]
- [[wiki/dms90/实体/MongoDB]]
