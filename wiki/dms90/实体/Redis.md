---
type: entity
tags: [缓存, 会话存储, 现场问题]
sources:
  - 20240624 dms 界面打开报错
  - 20250122-portal打开dms报错Access is denied
created: 2024-06-24
updated: 2026-04-20
---

# Redis

> DMS 使用 Redis 存储 Session 信息，与 Portal 共享实现单点登录。

## 在现场问题中的角色

### Session 数据格式错误

- [[wiki/dms90/来源/2024-06-24 DMS界面打开报错]]：Portal 反序列化 Redis 中信息时失败，session 数据格式不是 JSON

### RBAC 校验问题排除

- [[wiki/dms90/来源/2025-01-22 Portal打开DMS报错AccessDenied]]：403 错误排除 Redis 配置问题，实际是 RBAC 配置导致

## 关键要点

- Redis 中 session 数据必须为 JSON 格式
- session 数据被污染会导致反序列化失败
- Portal 通过 Redis 共享 session 实现 SSO

## 相关实体

- [[wiki/dms90/来源/2026-04-19 DMS项目概览|DMS]]
- [[wiki/dms90/实体/Portal]]

## 关联概念

- [[wiki/dms90/概念/缓存架构|缓存架构]]
