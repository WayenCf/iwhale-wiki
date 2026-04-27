---
type: source
tags: [现场问题, 批量处理, task_record, running状态, JasperReports死循环, 缅甸MPT]
sources: [20240823-MPT批量xml上传接口running.md]
created: 2024-08-23
updated: 2026-04-20
---


# MPT批量XML上传接口Running

## 问题现象

现场业务侧提示文件找不到，批量生产模版接口卡住不执行。

## 问题分析

定位到 `DMS_TASK_RECORD` 表状态为 `running` 卡住。

### 批量处理流程

1. 业务侧生成模版 XML 文件
2. 上传到与 [[wiki/dms90/来源/2026-04-19 DMS项目概览|DMS]] 主机共享目录下
3. [[wiki/dms90/来源/2026-04-19 DMS项目概览|DMS]] 批量处理逻辑扫描共享目录
4. 移动共享目录 XML 到 workspace 待处理目录下
5. 删除共享目录 XML
6. 代码记录 `DMS_TASK_RECORD` 表状态为 `running`
7. 代码获取 XML 文件并调用 make 生成模版文档接口
8. 修改记录 `DMS_TASK_RECORD` 表状态为 `success`

## 问题原因

- 模版 `CUST_BILL_ADDR` 字段内容超长
- 导致后台 `JRBaseFiller.fillf` 方法执行 waiting 死循环
- 只有 debug 日志可以看到，现场未查到报错

## 解决方案

1. 缩减 `CUST_BILL_ADDR` 字段内容超长，或模版设计处进行调整
2. 删除 `task_record` 表状态为 `running` 的记录
3. 重启 [[wiki/dms90/来源/2026-04-19 DMS项目概览|DMS]] 服务
4. 业务前台重新操作

## 关键要点

- `DMS_TASK_RECORD` 状态卡在 `running` 是常见批量处理问题
- 字段超长会导致 [[wiki/dms90/实体/JasperReports]] 死循环
- 死循环不一定会产生可见错误日志

## 相关实体

- 缅甸MPT
- [[wiki/dms90/来源/2026-04-19 DMS项目概览|DMS]]
- [[wiki/dms90/实体/JasperReports]]
- XML
- 批量处理