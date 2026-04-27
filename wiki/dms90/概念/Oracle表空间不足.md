---
type: concept
tags: [数据库, 表空间, Oracle, 现场问题]
sources:
  - 20240805-TM现场DMS问题unable to extend table FTF.DMS_DOCUMENT by 128 in tablespace TAB_FTF
  - 20240522-DMS模版小飞机推送和push all报错
created: 2024-05-22
updated: 2026-04-20
---

# Oracle表空间不足

> Oracle 表空间满时无法扩展表，导致 DMS 写入操作失败。

## 在现场问题中的角色

### DMS_DOCUMENT 表空间不足

- [[wiki/dms90/来源/2024-08-05 TM现场Oracle表空间不足]]：`TAB_FTF` 表空间不足，`unable to extend table FTF.DMS_DOCUMENT by 128`

### 模版推送失败

- [[wiki/dms90/来源/2024-05-22 DMS模版推送和PushAll报错]]：`unable to extend table FTF.DMS_RELEASE_RESOURCE by 1024 in tablespace TAB_FTF`

## 排查步骤

1. 查找表空间使用率
2. 查找可清理的大表（如 `poll_trigger_monitor`）
3. 执行 `TRUNCATE` 清理
4. 若不够则扩容表空间

## 常见可清理的表

| 表名 | 说明 | 清理方式 |
|------|------|---------|
| `poll_trigger_monitor` | 监控记录 | `TRUNCATE TABLE` |
| `DMS_LOG` | 操作日志 | 按日期删除 |

## 关键要点

- 表空间不足是 Oracle 环境常见问题
- 监控记录表可安全清理
- 需定期监控表空间使用率
- `ORA-01653` 错误码对应表空间不足

## 相关实体

- [[wiki/dms90/来源/2026-04-19 DMS项目概览|DMS]]
- [[wiki/dms90/实体/Oracle]]
