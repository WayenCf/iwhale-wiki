---
type: concept
tags: [兼容性, JDK, 升级, 现场问题]
sources:
  - 20250402-JDK17之后默认不保存方法参数名称引发-paramters报错
  - 20240312-格鲁项目dms垃圾回收启动失败Could not create the Java Virtual Machine
created: 2025-04-02
updated: 2026-04-20
---

# JDK版本兼容性

> JDK 版本升级后 DMS 的兼容性问题，主要涉及参数名称保留和垃圾收集器配置。

## 在现场问题中的角色

### JDK 17 方法参数名称

- [[wiki/dms90/来源/2025-04-02 JDK17方法参数名称报错]]：JDK 17 默认不保留方法参数名称，`@PathVariable` 不指定参数名报错
- 解决：全局搜索 `@PathVariable` 添加参数名称

### 垃圾收集器配置冲突

- [[wiki/dms90/来源/2024-03-12 格鲁项目DMS垃圾回收启动失败]]：基础镜像 JDK 默认 GC 与配置冲突
- 解决：使用 `COVER_OPTS` 覆盖

## 升级注意事项

### JDK 8 → JDK 17

| 变更项 | 影响 | 解决方案 |
|--------|------|---------|
| 方法参数名称不保留 | `@PathVariable`/`@RequestParam` 报错 | 显式指定参数名或添加 `-parameters` 编译参数 |
| 默认 GC 变为 G1 | CMS 不再支持 | 使用 `COVER_OPTS` 配置 |
| 模块系统 | 反射访问限制 | 添加 `--add-opens` 参数 |

## 关键要点

- JDK 17 升级需全面测试 DMS 功能
- `@PathVariable` 必须显式指定参数名
- 可通过编译参数 `-parameters` 保留参数名称
- 基础镜像版本需统一管理

## 相关实体

- [[wiki/dms90/来源/2026-04-19 DMS项目概览|DMS]]
- [[wiki/dms90/概念/JDK版本兼容性]]

## 关联概念

- [[wiki/dms90/概念/G1GC垃圾回收器|G1GC 垃圾回收器]]
