---
title: Sentinel
type: entity
created: 2026-04-20
updated: 2026-04-20
tags: [组件, 流控, 熔断, 限流]
sources: []
---

# Sentinel

> Alibaba Sentinel 流控组件，为 DMS 提供熔断降级和并发控制能力，保护存储层和上传/下载接口不被过载击穿。

## 在 DMS 中的角色

| 方面 | 详情 |
|------|------|
| 来源 | Alibaba Sentinel |
| 用途 | 存储层读写熔断 + 上传/下载并发限流 |
| 集成模块 | dms-common（规则定义）、dms-repo-mongodb（存储流控）、dms-web（上传/下载限流） |

## 熔断规则（dms-common / metric 包）

### 配置属性（MetricProperties，前缀 `ftf.dms.metric`）

| 属性 | 默认值 | 说明 |
|------|--------|------|
| enabled | false | 是否启用熔断 |
| readTimeWindow | 60s | 读熔断时间窗口 |
| readFailureThreshold | 200 | 读熔断连续异常次数 |
| readRt | 30000ms | 读熔断平均响应时间 |
| writeTimeWindow | 60s | 写熔断时间窗口 |
| writeFailureThreshold | 100 | 写熔断连续异常次数 |
| writeRt | 60000ms | 写熔断平均响应时间 |

### 4 条 DegradeRule

| 规则 | 资源名 | 策略 | 阈值 | 时间窗口 |
|------|--------|------|------|---------|
| writeRule | `dms_write_storage` | 异常数量 | 100 | 60s |
| rtWriteRule | `dms_write_storage` | 平均响应时间 | 60000ms | 60s |
| readRule | `dms_read_storage` | 异常数量 | 200 | 60s |
| rtReadRule | `dms_read_storage` | 平均响应时间 | 30000ms | 60s |

### MetricConfiguration

- @PostConstruct 加载：若 `metricProperties.enabled == true`，调用 MetricUtil.loadDegradeRules()
- dms-web 的 MetricController 提供 `/metric/start` 和 `/metric/stop` 端点动态控制

## 存储层流控（dms-repo-mongodb）

### MongoDao<T> 中的 Sentinel 集成

- **写操作**：`SphU.entry(DMS_WRITE_STORAGE_METRIC)` 包裹 create/update/delete
- **读操作**：`SphU.entry(DMS_READ_STORAGE_METRIC)` 包裹 retrieve
- **异常映射**：
  - DegradeException → StorageDegradeException（熔断降级）
  - BlockException → MetricBlockException（流控阻断）
  - 手动增加异常：MetricEntryUtil.increaseBizException()

## 上传/下载限流（dms-web）

### SentinelConfig

- CommandLineRunner 启动后初始化线程级流控规则：
  - `dms_upload_task`：限制上传并发数
  - `dms_download_task`：限制下载并发数
- 实际限制 = min(配置线程数, Tomcat 线程数 * 40%)

## 指标采集

- DMSMetricInfoSender 实现 MetricSender 接口
- 采集 pass_qps / success_qps / exception_qps / block_qps / timestamp / rt
- 通过 ZSmartLogger.stat() 输出

## 与自研 flow 包的关系

DMS 存在**双限流体系**：
- **自研 flow 包**：秒级流量统计写 CSV，用于**监控**
- **Sentinel**：异常数/RT 熔断，用于**保护**

两者互不冲突，flow 包偏观察，Sentinel 偏防护。

## 关联概念

- [[wiki/dms90/概念/限流机制|限流机制]]
- [[wiki/dms90/概念/异常体系|异常体系]]
- [[wiki/dms90/概念/缓存架构|缓存架构]]

## 关联页面

- [[wiki/dms90/来源/2026-04-20 代码分析-dms-common|代码分析：dms-common]]
- [[wiki/dms90/来源/2026-04-20 代码分析-dms-web|代码分析：dms-web]]
- [[wiki/dms90/来源/2026-04-20 代码分析-dms-repo-mongodb|代码分析：dms-repo-mongodb]]
