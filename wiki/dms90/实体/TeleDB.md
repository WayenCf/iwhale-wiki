---
type: entity
tags: [数据库, 关系型数据库, 集群, 现场问题]
sources:
  - 20241012-PTO配置环境导入模板报错
  - 20250310-模版导入问题汇总
created: 2024-10-12
updated: 2026-04-20
---

# TeleDB

> 分布式关系型数据库，部分现场环境使用 TeleDB 替代 Oracle 作为 DMS 后端数据库。

## 在现场问题中的角色

### 数据库连接中断

- [[wiki/dms90/来源/2024-10-12 PTO配置环境导入模板报错]]：服务端主动关闭连接，切换集群节点解决
- [[wiki/dms90/来源/2025-03-10 模版导入问题汇总]]：TeleDB 配置的节点挂了，切换到另一个节点

## 关键要点

- TeleDB 集群部署时需注意连接节点可用性
- 节点故障时需切换到集群中其他节点
- 连接中断报错信息：`The last packet successfully received from the server was xxx milliseconds ago`

## 常见故障模式

1. **节点故障** → 连接中断 → 切换到集群其他节点
2. **服务端主动关闭连接** → EOFException → 重启数据库或切换节点

## 相关实体

- [[wiki/dms90/来源/2026-04-19 DMS项目概览|DMS]]
- [[wiki/dms90/实体/Oracle]]
