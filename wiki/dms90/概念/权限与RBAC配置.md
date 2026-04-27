---
type: concept
tags: [安全, RBAC, 越权, 菜单, 权限]
sources:
  - 20240328-DMS菜单权限优化
  - 20250122-portal打开dms报错Access is denied
  - 20241205-TM安全测试报告分析
created: 2024-03-28
updated: 2026-04-21
---

# 权限与RBAC配置

> DMS 的菜单权限控制和 RBAC（基于角色的访问控制）配置。

## 在现场问题中的角色

### Admin 菜单权限

- [[wiki/dms90/来源/2024-03-28 DMS菜单权限优化]]：Admin 用户需显示 Configuration 菜单，非 Admin 用户不显示

### RBAC 越权校验

- [[wiki/dms90/来源/2025-01-22 Portal打开DMS报错AccessDenied]]：`enable-rbac=true` 导致 403
- 解决：`app.security.enable-rbac=false`

### 安全测试

- [[wiki/dms90/来源/2024-12-05 TM安全测试报告分析]]：纵向越权漏洞测试结果

## 关键配置

| 配置项 | 说明 | 建议值 |
|--------|------|--------|
| `app.security.enable-rbac` | RBAC 纵向越权校验 | 视需求关闭 |
| `app.security.develop-mode` | 开发者模式 | 生产环境 false |
| `app.security.csrf-url-skip` | CSRF 跳过规则 | 必须配置 |

## 角色与菜单映射

| 角色 | 可见菜单 |
|------|---------|
| ROLE_ADMIN | Home, Template, Label, Configuration, Certificate, Inquiry&Statistic |
| 非 ADMIN | Home, Template, Label, Certificate, Inquiry&Statistic |

## DMS_USER 表中的角色字段

- `USER_ROLE`：VARCHAR2，存储用户角色信息
- `ENABLED`：CHAR，控制用户是否启用
- `TENANT`：VARCHAR2，外键引用 `DMS_TENANT.ID`，实现租户级权限隔离
- `CRT`：VARCHAR2，记录最后修改用户 ID，用于审计追踪

> 详见 [[wiki/dms90/来源/2026-04-21 DMS关键表结构|DMS 关键表结构]]。

## 关键要点

- RBAC 校验过严可能导致 403
- 开发者模式必须在生产环境关闭
- 菜单列表由后端根据角色动态返回

## 相关实体

- [[wiki/dms90/来源/2026-04-19 DMS项目概览|DMS]]
- [[wiki/dms90/实体/Portal]]

## 关联概念

- [[wiki/dms90/概念/安全加固|安全加固]]
