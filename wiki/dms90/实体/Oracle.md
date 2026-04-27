---
type: entity
tags: [数据库, 关系型数据库, 现场问题]
sources:
  - 20240408-马来TM项目-SIT环境创建客户上传文件dms报错DMS 数据库主键冲突问题
  - 20240530-日志大量insert超时报错
  - 20240805-TM现场DMS问题unable to extend table FTF.DMS_DOCUMENT by 128 in tablespace TAB_FTF
  - 20250310-模版导入问题汇总
  - 20250926-DMS序列问题
  - 20250610-新电生产环境发布模版报错-序列问题
- 2026-04-21-DMS关键表结构
created: 2024-04-08
updated: 2026-04-21
---

# Oracle

> DMS 关系型数据库后端选项之一，用于存储元数据、操作日志、模版资源等信息。

## 在现场问题中的角色

### 序列与主键冲突

- [[wiki/dms90/来源/2024-04-08 马来TM数据库主键冲突问题]]：数据导入导致序列缓存未更新，重启应用解决
- [[wiki/dms90/来源/2025-09-26 DMS序列问题]]：[[wiki/dms90/来源/2026-04-19 DMS项目概览|DMS]] 内部缓存序列值，修改数据库后需重启刷新
- [[wiki/dms90/来源/2025-06-10 新电生产环境发布模版序列问题]]：数据迁移未同步 `tfm_sequences` 表

### 表空间不足

- [[wiki/dms90/来源/2024-08-05 TM现场Oracle表空间不足]]：`TAB_FTF` 表空间不足，`poll_trigger_monitor` 表过大
- [[wiki/dms90/来源/2024-05-22 DMS模版推送和PushAll报错]]：`TAB_FTF` 表空间不足导致推送失败

### CLOB 字段问题

- [[wiki/dms90/来源/2025-03-10 模版导入问题汇总]]：`DMS_RESOURCE.DEPENDENCIES` 字段超长，ORA-22275 LOB 定位符无效

### 性能问题

- [[wiki/dms90/来源/2024-05-30 日志大量insert超时报错]]：大量 insert 操作超时

## 关键表

| 表名 | 主键 | 说明 | 常见问题 |
|------|------|------|---------|
| `DMS_USER` | ID (VARCHAR2) | 用户账号与角色，TENANT 外键引用 DMS_TENANT | 用户不存在导致模版无法使用 |
| `DMS_TENANT` | ID (VARCHAR2) | 租户与存储配额（TOTAL_SIZE / USED_SIZE） | 租户配置错误导致多租户异常 |
| `DMS_RESOURCE` | ID (VARCHAR) | 模版资源元数据，ID 即模版路径 | CLOB 字段超长（ORA-22275） |
| `DMS_RELEASE` | ID (VARCHAR2) | 发布记录，USER_ID → DMS_USER.ID | 序列不一致 |
| `DMS_RELEASE_RESOURCE` | (RELEASE_ID, ID) 联合主键 | 发布资源明细，RELEASE_ID → DMS_RELEASE.ID | — |
| `DMS_FILE` | — | 文件记录 | 序列冲突 |
| `DMS_TASK_RECORD` | — | 批量任务记录 | 状态卡在 running |
| `DMS_LOG` | — | 操作日志 | insert 超时 |
| `tfm_sequences` | — | 序列管理 | 迁移未同步 |
| `poll_trigger_monitor` | — | 监控记录 | 表过大需清理 |

> 详细的表字段定义和关联关系见 [[wiki/dms90/来源/2026-04-21 DMS关键表结构]]。

## 常见故障模式

1. **序列不一致** → 主键冲突 → 重启/手动调整序列
2. **表空间满** → ORA-01653 → 清理大表/扩容
3. **CLOB 字段超长** → ORA-22275 → 手动处理

## 相关实体

- [[wiki/dms90/来源/2026-04-19 DMS项目概览|DMS]]
- [[wiki/dms90/实体/TeleDB]]
