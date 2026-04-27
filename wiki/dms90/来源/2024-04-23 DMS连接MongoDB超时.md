---
type: source
tags: [现场问题, MongoDB超时, 连接超时, 文件系统满, MongoDB服务]
sources: [20240423-DMS连接MongDB超时Timed out after 30000 ms.md]
created: 2024-04-23
updated: 2026-04-20
---


# DMS连接MongoDB超时

## 问题现象

[[wiki/dms90/来源/2026-04-19 DMS项目概览|DMS]] 连接 [[wiki/dms90/实体/MongoDB]] 超时报错 `Timed out after 30000 ms`。

## 问题跟踪

1. 进入 [[wiki/dms90/来源/2026-04-19 DMS项目概览|DMS]] 主机，检查配置文件，确认 [[wiki/dms90/实体/MongoDB]] 配置连接正确
2. `telnet` [[wiki/dms90/实体/MongoDB]] 连接地址不通
3. 继续跟踪 [[wiki/dms90/实体/MongoDB]] 服务，发现 [[wiki/dms90/实体/MongoDB]] 文件系统满了

## 问题原因

[[wiki/dms90/实体/MongoDB]] 文件系统满了，导致 [[wiki/dms90/实体/MongoDB]] 服务挂了。

## 解决方案

清理文件系统空间后重启 [[wiki/dms90/实体/MongoDB]] 服务解决。

## 关键要点

- [[wiki/dms90/实体/MongoDB]] 文件系统满会导致服务不可用
- 30000ms 超时是 [[wiki/dms90/来源/2026-04-19 DMS项目概览|DMS]] 默认连接超时时间
- 需要定期监控 [[wiki/dms90/实体/MongoDB]] 磁盘空间

## 相关实体

- [[wiki/dms90/来源/2026-04-19 DMS项目概览|DMS]]
- [[wiki/dms90/实体/MongoDB]]
- 连接超时
