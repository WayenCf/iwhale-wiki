---
type: concept
tags: [性能, 垃圾回收, JVM, 配置]
sources:
  - 20240312-格鲁项目dms垃圾回收启动失败Could not create the Java Virtual Machine
  - 20260107-印尼账单模版字体展示不全
created: 2024-03-12
updated: 2026-04-20
---

# G1GC垃圾回收器

> G1 (Garbage-First) 垃圾收集器，JDK 9+ 默认垃圾收集器，适合大堆内存应用。

## 在现场问题中的角色

### 垃圾收集器冲突

- [[wiki/dms90/来源/2024-03-12 格鲁项目DMS垃圾回收启动失败]]：基础镜像 JDK 默认带 ConcMarkSweepGC，环境变量配置 G1GC，两个同时配置导致 JVM 启动失败
- 解决方案：使用 `COVER_OPTS=-XX:+UseG1GC` 强制覆盖

## 配置方式

### 不推荐：JAVA_OPTS 中直接配置

```bash
JAVA_OPTS=-XX:+UseG1GC  # 可能与基础镜像默认GC冲突
```

### 推荐：使用 COVER_OPTS 覆盖

```bash
JAVA_OPTS=-Xms8192m -Xmx8192m ...  # 不含GC配置
COVER_OPTS=-XX:+UseG1GC  # 强制覆盖GC配置
```

## 关键要点

- G1GC 是 JDK 9+ 默认 GC，低版本 JDK 默认使用 CMS
- 不允许同时配置两个垃圾收集器
- `COVER_OPTS` 环境变量可以覆盖基础镜像的 GC 配置
- G1GC 适合大堆内存（>4GB）场景

## 相关实体

- [[wiki/dms90/来源/2026-04-19 DMS项目概览|DMS]]
- [[wiki/dms90/概念/JDK版本兼容性]]
