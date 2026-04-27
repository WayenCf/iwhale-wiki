---
type: entity
tags: [平台, 网关, 单点登录, 现场问题]
sources:
  - 20240328-DMS菜单权限优化
  - 20240624 dms 界面打开报错
  - 20250122-portal打开dms报错Access is denied
  - 20240408-DMS配置问题解答
created: 2024-03-28
updated: 2026-04-22
---

# Portal

> iWhaleCloud 统一门户平台，DMS 通过 Portal 实现单点登录和菜单集成。

## 在现场问题中的角色

### 菜单权限控制

- [[wiki/dms90/来源/2024-03-28 DMS菜单权限优化]]：Admin 用户需显示 Configuration 菜单
- 菜单列表由后端 `/menu/list` 接口根据用户角色返回

### Session 反序列化失败

- [[wiki/dms90/来源/2024-06-24 DMS界面打开报错]]：Portal 反序列化 Redis 中 session 信息失败
- Session 数据格式不是 JSON 导致解析报错

### RBAC 越权校验

- [[wiki/dms90/来源/2025-01-22 Portal打开DMS报错AccessDenied]]：`enable-rbac` 开启导致 403
- 解决：`app.security.enable-rbac=false`

### 安全配置

- [[wiki/dms90/来源/2024-04-08 DMS配置问题解答]]：`app.security.develop-mode=false` 关闭开发者模式

### HTTPS 协议不匹配

- [[wiki/dms90/来源/2026-04-22 业务侧调用DMS生成PDF返回400]]：业务侧通过 Portal 调用 DMS 生成 PDF，URL 使用 `http://` 访问 443 端口（HTTPS），返回 400

## 关键配置

| 配置项 | 说明 |
|--------|------|
| `app.security.develop-mode` | 开发者模式开关，生产环境需关闭 |
| `app.security.enable-rbac` | RBAC 纵向越权校验开关 |

## 常见故障模式

1. **Session 格式错误** → 反序列化失败 → 界面报错
2. **RBAC 校验过严** → 403 Access Denied
3. **开发者模式未关** → 未授权访问漏洞
4. **HTTPS 协议不匹配** → 400 Bad Request（HTTP 请求 HTTPS 端口）

## 关联概念

- [[wiki/dms90/概念/HTTPS协议配置|HTTPS 协议配置]]
- [[wiki/dms90/概念/权限与RBAC配置|权限与 RBAC 配置]]

## 相关实体

- [[wiki/dms90/来源/2026-04-19 DMS项目概览|DMS]]
- [[wiki/dms90/实体/Redis]]
