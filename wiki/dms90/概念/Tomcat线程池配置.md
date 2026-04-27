---
type: concept
tags: [性能, Tomcat, 线程池, 并发, 配置]
sources:
  - 20240321- 马来UM-DMS-tomcat-web容器参数调优
  - 20240402-格鲁生产环境DMS出账较慢
created: 2024-03-21
updated: 2026-04-20
---

# Tomcat线程池配置

> Spring Boot 内嵌 Tomcat 的线程池和连接参数配置，直接影响 DMS 的并发处理能力。

## 在现场问题中的角色

### 线程阻塞导致出账中断

- [[wiki/dms90/来源/2024-03-21 马来UM Tomcat参数调优]]：所有工作线程阻塞在子报表元素组装环节，客户端请求无法处理

### 出账性能调优

- [[wiki/dms90/来源/2024-04-02 格鲁生产环境DMS出账较慢]]：出账性能下降，部分因 Tomcat 处理能力不足

## 关键配置项

| 配置项 | 说明 | 默认值 | 建议值 |
|--------|------|--------|--------|
| `server.tomcat.threads.max` | 最大工作线程数 | 200 | 200（不建议增加） |
| `server.tomcat.max-connections` | 最大连接数 | 8192 | 200（快速失败） |
| `server.tomcat.accept-count` | 等待队列长度 | 100 | 1（快速失败） |

## 调优策略

### 快速失败模式（推荐）

```properties
server.tomcat.threads.max=200
server.tomcat.max-connections=200
server.tomcat.accept-count=1
```

- 当所有线程忙时，新请求立即拒绝而不是排队等待
- 避免请求积压导致系统雪崩
- 需配合网关的负载均衡和重试机制

### 吞吐量优先模式

```properties
server.tomcat.threads.max=400
server.tomcat.max-connections=8192
server.tomcat.accept-count=100
```

- 适合请求处理时间短的场景
- 注意线程数过多会增加上下文切换开销

## 关键要点

- 降低 `max-connections` 和 `accept-count` 实现快速失败
- 增加线程数不建议超过 200，会引发其他问题
- 需配合网关负载均衡使用

## 相关实体

- [[wiki/dms90/来源/2026-04-19 DMS项目概览|DMS]]
- [[wiki/dms90/概念/Tomcat线程池配置|Tomcat]]
- [[wiki/dms90/实体/Nginx|Nginx]]
