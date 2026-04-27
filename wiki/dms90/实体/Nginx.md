---
type: entity
tags: [Web服务器, 反向代理, 负载均衡, 现场问题]
sources:
  - 20250605-TM现场业务侧调用dms请求失败
  - 20240321- 马来UM-DMS-tomcat-web容器参数调优
created: 2025-06-05
updated: 2026-04-22
---

# Nginx

> 反向代理服务器，在 DMS 部署架构中用于请求转发和负载均衡。

## 在现场问题中的角色

### Reload 导致请求中断

- [[wiki/dms90/来源/2025-06-05 TM现场业务侧调用DMS请求失败]]：nginx 配置类型为模版刷新，应用重新部署时 nginx 执行 reload，导致短暂请求中断

### 阿富汗 HTTPS 转发

- [[wiki/dms90/来源/2025-09-23 阿富汗ET现场容灾切换]]：阿富汗主机部署通过 nginx 转发 DMS

### HTTPS 协议不匹配

- [[wiki/dms90/来源/2026-04-22 业务侧调用DMS生成PDF返回400]]：Nginx 监听 443 端口（HTTPS），但业务侧用 HTTP 协议请求，返回 400

## 关键要点

- nginx reload 会短暂中断请求
- 模版刷新类型网关在应用部署时自动触发 reload
- 相同网关下的应用部署需协调时间窗口
- 可考虑使用 nginx upstream 的热更新机制
- **443 端口必须使用 HTTPS 协议**，HTTP 请求 HTTPS 端口返回 400 Bad Request

## 关联概念

- [[wiki/dms90/概念/HTTPS协议配置|HTTPS 协议配置]]
- [[wiki/dms90/概念/Tomcat线程池配置|Tomcat]]
