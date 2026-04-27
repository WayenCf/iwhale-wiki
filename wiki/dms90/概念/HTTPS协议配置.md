---
type: concept
tags: [HTTPS, SSL, 协议配置, 现场问题]
sources:
  - 20250923-阿富汗ET现场容灾切换
  - 2026-03-18-业务侧调用DMS生成PDF返回400
created: 2026-04-22
updated: 2026-04-22
---

# HTTPS 协议配置

> DMS 部署中 HTTPS 协议的配置与常见协议不匹配问题。当 DMS 或 Portal 启用 HTTPS（端口 443）后，所有调用方必须同步使用 HTTPS 协议，否则请求失败。

## 在现场问题中的角色

### HTTP 请求 HTTPS 端口 → 400 Bad Request

- [[wiki/dms90/来源/2026-04-22 业务侧调用DMS生成PDF返回400]]：`DmsRestFulUtil` 构造 URL 使用 `http://`，但目标端口 443 是 HTTPS 端口，服务端返回 400 Bad Request [no body]

### DMS 切换 HTTPS 后调用方未同步

- [[wiki/dms90/来源/2025-09-23 阿富汗ET现场容灾切换]]：DMS 配置 HTTPS 后，CRM 等业务侧未同步修改协议，报 401 错误；修改为 HTTPS 后又因未忽略证书验证仍然失败

## 协议与端口对应关系

| 端口 | 正确协议 | 错误协议 | 错误表现 |
|------|---------|---------|---------|
| 80 / 8080 | `http://` | `https://` | 连接被拒绝或 SSL 握手失败 |
| 443 / 8443 | `https://` | `http://` | **400 Bad Request [no body]** |

> **核心规则**：协议必须与端口匹配。HTTP 请求 HTTPS 端口时，服务端收到的是未加密的 HTTP 字节流，无法按 TLS 协议解析，直接返回 400。

## DMS HTTPS 配置

```properties
server.port=8099
server.ssl.key-store=/app/ZSMART_HOME/keystore_2.p12
server.ssl.enabled=true
server.ssl.key-store-password=<密码>
server.ssl.key-alias=zcm
```

## 切换 HTTPS 后的检查清单

| # | 检查项 | 说明 |
|---|--------|------|
| 1 | DMS SSL 配置 | `server.ssl.enabled=true`，keystore 路径和密码正确 |
| 2 | 所有调用方协议同步 | `http://` → `https://`，包括 `DmsRestFulUtil`、CRM、Billing 等 |
| 3 | 调用方忽略证书验证 | 自签名证书需配置 `trustAllCertificates` 或导入 CA |
| 4 | Nginx 转发配置 | `proxy_pass https://`，如使用 Nginx 做反向代理 |
| 5 | Portal 转发配置 | Portal 的 DMS 代理地址需同步修改协议 |

## 常见故障模式

1. **协议不匹配** → 400 Bad Request [no body]（HTTP 请求 HTTPS 端口）
2. **证书未信任** → 401 Unauthorized 或 SSLHandshakeException
3. **调用方未同步** → 切换 HTTPS 后部分业务侧仍用 HTTP 调用

## 故障排查步骤

1. 确认目标服务端口是 HTTP 还是 HTTPS（443/8443 通常为 HTTPS）
2. 检查调用方构造的 URL 协议前缀是否匹配
3. 如果是 HTTPS，确认调用方是否信任服务端证书
4. 检查 Nginx/Portal 转发配置中的 `proxy_pass` 协议

## 相关实体

- [[wiki/dms90/实体/Nginx]]
- [[wiki/dms90/实体/Portal]]
- [[wiki/dms90/来源/2026-04-19 DMS项目概览|DMS]]

## 关联概念

- [[wiki/dms90/概念/安全加固|安全加固]]
- [[wiki/dms90/概念/离线账单生成|离线账单生成]]