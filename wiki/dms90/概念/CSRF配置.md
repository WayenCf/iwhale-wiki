---
type: concept
tags: [安全, CSRF, 配置, 现场问题]
sources:
  - 20240408-DMS配置问题解答
  - 20250310-模版导入问题汇总
created: 2024-04-08
updated: 2026-04-20
---

# CSRF配置

> Cross-Site Request Forgery 跨站请求伪造防护，DMS 通过 Spring Security CSRF Filter 实现。

## 在现场问题中的角色

### 开发者模式未关闭

- [[wiki/dms90/来源/2024-04-08 DMS配置问题解答]]：`app.security.develop-mode=false` 关闭开发者模式后需配置 CSRF 跳过规则

### 模版导入被拦截

- [[wiki/dms90/来源/2025-03-10 模版导入问题汇总]]：未配置 `csrf-url-skip` 导致模版导入请求被 CSRF 校验拦截

## 关键配置

```properties
# CSRF 跳过规则
app.security.csrf-url-skip=/doc.*|/release.*|/prerelease.*|/publish.*|/delegate.*|/doc.*|/file.*|/folder.*|/material.*|/search.*|/resources.*|/label.*|/metadata.*|/dms/services.*|/dms/rest_v2.*|/dms/j_spring_security_check.*|/dms/crt.*|/crt.*

# 开发者模式（生产环境必须关闭）
app.security.develop-mode=false
```

## 关键要点

- 生产环境必须关闭开发者模式
- 关闭后需配置 `csrf-url-skip` 跳过 DMS 内部接口
- `DMSCsrfFilter` 继承 Spring Security 的 `CsrfFilter`

## 相关实体

- [[wiki/dms90/来源/2026-04-19 DMS项目概览|DMS]]
- 安全配置
