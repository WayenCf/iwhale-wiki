---
type: source
tags: [现场问题, Portal访问, AccessDenied, RBAC越权校验, 菜单权限]
sources: [20250122-portal打开dms报错Access is denied.md]
created: 2025-01-22
updated: 2026-04-20
---


# Portal打开DMS报错Access is denied

## 问题现象

从 [[wiki/dms90/实体/Portal]] 打开 [[wiki/dms90/来源/2026-04-19 DMS项目概览|DMS]]，接口返回 403 Access is denied。

## 问题分析

- 接口访问 403，排除 [[wiki/dms90/实体/Redis]] 配置错误问题
- 日志看到 `menu list is empty`

## 问题解决

配置以下参数关闭纵向越权校验：

```properties
app.security.enable-rbac=false
```

## 关键要点

- `enable-rbac=false` 关闭纵向越权校验
- 403 错误不一定是 Redis 问题，也可能是权限校验
- 菜单列表为空时优先检查 RBAC 配置

## 相关实体

- [[wiki/dms90/来源/2026-04-19 DMS项目概览|DMS]]
- [[wiki/dms90/实体/Portal]]
- [[wiki/dms90/概念/权限与RBAC配置]]