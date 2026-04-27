---
type: concept
tags: [使用指南, 工作流, 问题分析, 代码重构]
sources: []
created: 2026-04-20
updated: 2026-04-22
---

# DMS90 知识库高效使用指南

> 面向开发、运维、重构场景的知识库使用方法论

---

## 📖 知识库结构速览

```
dms90 知识库（97 页）
├── 来源（56 页）— 项目文档、现场问题、代码分析
├── 实体（13 页）— 技术组件：MongoDB、JasperReports、Sentinel 等
├── 概念（28 页）— 设计机制：分表、多租户、限流、缓存等
└── 检索指南（2 页）— 现场问题快速检索、使用指南
```

---

## 🎯 场景一：现场问题分析

### 工作流程

```
现场问题报错
    ↓
① 在 Obsidian 中 Ctrl+Shift+F 搜索错误关键字
    ↓
② 命中？是 → 进入 ③ | 否 → 进入 ④
    ↓
③ 打开对应现场问题来源页 → 查看解决方案 → 应用
    ↓
④ 打开 [[wiki/dms90/概念/现场问题快速检索指南]]
    ↓
⑤ 按速查表定位到相关概念页
    ↓
⑥ 概念页 → 查看所有相关来源页 → 综合分析
    ↓
⑦ 解决后，如为新问题 → 摄入知识库
```

### 实操示例

**问题**：生产环境报 `Timed out after 30000 ms awating connection`

**步骤**：
1. `Ctrl+Shift+F` → 搜索 `"Timed out after 30000"`
2. 找到 [[wiki/dms90/来源/2024-04-23 DMS连接MongoDB超时]]
3. 查看解决方案：MongoDB 文件系统满导致服务挂
4. 定位到 [[wiki/dms90/概念/MongoDB连接超时]] → 查看该问题的所有现场案例
5. 检查生产 MongoDB 文件系统 → 清理空间 → 重启服务

---

## 🔧 场景二：代码重构

### 工作流程

```
重构需求
    ↓
① 定位重构涉及的模块/功能
    ↓
② 阅读对应代码分析来源页
    ↓
③ 理解设计意图和关键机制
    ↓
④ 查看相关概念页（跨模块影响）
    ↓
⑤ 检查现场问题（潜在风险）
    ↓
⑥ 制定重构方案并执行
```

### 重构类型与知识库路径

| 重构类型 | 首选页面 | 辅助页面 |
|---------|---------|---------|
| **修改存储逻辑** | [[wiki/dms90/来源/2026-04-20 代码分析-dms-repo-mongodb]] | [[wiki/dms90/概念/shardKey分表机制]] [[wiki/dms90/概念/多租户隔离]] |
| **新增 API 端点** | [[wiki/dms90/来源/2026-04-20 代码分析-dms-web]] | [[wiki/dms90/来源/2026-04-19 DocumentController接口逻辑梳理]] |
| **优化批量生成** | [[wiki/dms90/来源/2026-04-20 代码分析-dms-service-impl]] | [[wiki/dms90/概念/批量账单生成流程]] |
| **调整限流策略** | [[wiki/dms90/概念/限流机制]] | [[wiki/dms90/实体/Sentinel]] |
| **修改缓存机制** | [[wiki/dms90/概念/缓存架构]] | [[wiki/dms90/实体/Redis]] [[wiki/dms90/实体/Guava Cache]] |
| **调整多租户** | [[wiki/dms90/概念/多租户隔离]] | [[wiki/dms90/概念/实体继承体系]] |

### 实操示例：重构批量生成流程

**需求**：优化大 XML 文件处理性能

**步骤**：
1. **理解现状**：
   - 读 [[wiki/dms90/来源/2026-04-20 代码分析-dms-service-impl]] → 找到 `BatchDocumentServiceImpl`
   - 读 [[wiki/dms90/概念/批量账单生成流程]] → 理解完整流程

2. **识别瓶颈**：
   - 发现 XML 切割机制（`DivisibleXml`、`HeavyDocMakeTask`）
   - 读 [[wiki/dms90/来源/2024-08-23 MPT批量XML上传接口Running]] → 字段超长导致死循环

3. **查看设计**：
   - 读 `代码分析-dms-service-impl` → 切割配置 `CuttingProperties`
   - 三种失败策略：`throwException` / `goOnWithoutCutting` / `silent`

4. **制定方案**：
   - 调整 `maxMemoryUsage`、`failStrategy`
   - 参考现场问题避免字段超长

5. **验证影响**：
   - 检查 [[wiki/dms90/概念/离线账单生成]] → 确认不影响离线模式
   - 检查相关现场问题 → 避免引入已知问题

---

## 📚 场景三：快速学习理解

### 新人上手路径

```
第一天：宏观理解
├─ [[wiki/dms90/来源/2026-04-19 DMS项目概览]] → 整体架构
├─ [[wiki/dms90/概念/实体继承体系]] → 数据模型
└─ [[wiki/dms90/概念/双存储适配]] → 存储架构

第二天：核心机制
├─ [[wiki/dms90/概念/shardKey分表机制]] → 分表路由
├─ [[wiki/dms90/概念/多租户隔离]] → 多租户设计
└─ [[wiki/dms90/概念/限流机制]] → Sentinel 流控

第三天：代码深入
├─ [[wiki/dms90/来源/2026-04-20 代码分析-dms-common]] → 基础模块
├─ [[wiki/dms90/来源/2026-04-20 代码分析-dms-web]] → Web 层
└─ [[wiki/dms90/来源/2026-04-20 代码分析-dms-service-impl]] → 业务层

第四天：故障排查
└─ [[wiki/dms90/概念/现场问题快速检索指南]] → 学会检索方法
```

### 按"技术栈"学习

| 感兴趣的技术 | 入口页面 |
|-------------|---------|
| **MongoDB** | [[wiki/dms90/实体/MongoDB]] → 代码分析-dms-repo-mongodb |
| **JasperReports** | [[wiki/dms90/实体/JasperReports]] → 代码分析-dms-service-impl（PDF 签名） |
| **Spring Boot** | [[wiki/dms90/实体/Spring Boot]] → 代码分析-dms-web（安全配置） |
| **Sentinel** | [[wiki/dms90/实体/Sentinel]] → 限流机制 |
| **多租户设计** | [[wiki/dms90/概念/多租户隔离]] → 三级数据库隔离 |

---

## 🔍 场景四：影响分析

### 改动影响评估流程

```
代码改动
    ↓
① 识别改动的实体/概念
    ↓
② 打开对应实体/概念页
    ↓
③ 查看"关联的来源页"和"涉及的概念"
    ↓
④ 打开所有关联页面 → 理解影响范围
    ↓
⑤ 检查现场问题 → 识别潜在风险
    ↓
⑥ 制定测试和发布策略
```

### 示例：修改 shardKey 规则

**改动**：修改 shardKey 生成逻辑

**影响分析**：
1. 打开 [[wiki/dms90/概念/shardKey分表机制]]
2. 查看关联：
   - [[wiki/dms90/概念/文档ID生命周期]] → ID 解析
   - [[wiki/dms90/概念/多租户隔离]] → ID 格式
   - [[wiki/dms90/实体/MongoDB]] → 集合路由
3. 检查现场问题：
   - [[wiki/dms90/来源/2024-03-28 阿富汗下载PDF超出限制条数]] → 查询超限
   - [[wiki/dms90/来源/2026-04-20 离线上传文件不扫描]] → shardKey 缺失
4. 风险识别：
   - 历史数据 shardKey 不一致 → 需要数据迁移
   - 跨分表查询 → 需要调整查询逻辑

---

## 🛠️ 高效技巧

### 技巧 1：利用 Obsidian 图谱视图

1. 打开某个概念页（如 [[wiki/dms90/概念/shardKey分表机制]]）
2. 点击左上角"图谱"图标
3. 看到该概念的所有关联：
   - 绿色节点：来源页（现场问题、代码分析）
   - 蓝色节点：实体页（技术组件）
   - 紫色节点：概念页（相关机制）
4. 点击节点 → 快速跳转 → 理解关联关系

### 技巧 2：反向链接（Backlinks）

1. 打开任意页面
2. 查看左下角"反向链接"面板
3. 看到"哪些页面引用了当前页面"
4. 用途：
   - 查看某个实体在哪些现场问题中被提及
   - 查看某个概念在哪些代码模块中被使用

### 技巧 3：Dataview 高级查询

如果你安装了 Dataview 插件：

```dataview
# 查询所有关于 MongoDB 的现场问题
TABLE WITHOUT ID
  file.link AS "问题",
  dateformat(file.ctime, "yyyy-MM-dd") AS "日期"
FROM "wiki/dms90/来源"
WHERE contains(file.content, "MongoDB")
SORT file.ctime DESC
```

```dataview
# 查询所有提到某个概念的地方
TABLE WITHOUT ID
  file.link AS "页面"
FROM "wiki/dms90"
WHERE contains(file.content, "shardKey")
```

### 技巧 4：标签筛选

- 每个页面都有 `tags` 字段
- 常用标签：`#现场问题`、`#代码分析`、`#MongoDB`、`#JasperReports`
- Obsidian 左侧栏 → 搜索图标 → 按 `#` 筛选标签

### 技巧 5：大纲快速跳转

- 打开长页面（如代码分析）
- 左侧大纲显示所有章节
- 点击章节 → 快速跳转

---

## 📋 快速参考卡片

### 问题诊断卡片

| 症状 | 首选搜索 | 关键页面 |
|------|---------|---------|
| 连接超时 | `Timed out` | [[wiki/dms90/概念/MongoDB连接超时]] |
| 查询超限 | `exceeds the limit` | [[wiki/dms90/实体/MongoDB]] |
| 文件上传失败 | `upload` | [[wiki/dms90/概念/云盘权限问题]] |
| PDF 生成失败 | `PDF` | [[wiki/dms90/概念/字体渲染问题]] |
| 403 权限报错 | `Access is denied` | [[wiki/dms90/概念/权限与RBAC配置]] |
| 模版预览异常 | `预览` | [[wiki/dms90/概念/模版预览问题]] |
| 序列冲突 | `主键冲突` | [[wiki/dms90/概念/序列问题]] |
| JVM 启动失败 | `Could not create` | [[wiki/dms90/概念/G1GC垃圾回收器]] |

### 运维查询卡片

| 场景 | 首选查询语句 | 参考页面 |
|------|-------------|---------|
| 查询最新文档 | `db.getCollection("Document").find().sort({ "creationDate": -1 })` | [[wiki/dms90/概念/DMS MongoDB常用查询]] |
| 查询最新模版 | `db.getCollection("Resource").find().sort({ "creationDate": -1 })` | [[wiki/dms90/概念/DMS MongoDB常用查询]] |
| 查询模版是否存在 | `db.getCollection("Resource").findOne({ "fileName": "eRFpage.jasper" })` | [[wiki/dms90/概念/DMS MongoDB常用查询]] |
| 查询最新文件 | `db.getCollection("File-20263").find().sort({ "creationDate": -1 })` | [[wiki/dms90/概念/DMS MongoDB常用查询]] |
| 查询任务记录 | `db.getCollection("TaskRecord-20263").find().sort({ "creationDate": -1 })` | [[wiki/dms90/概念/DMS MongoDB常用查询]] |

### 重构风险评估卡片

| 改动范围 | 必读页面 | 风险等级 |
|---------|---------|---------|
| **ID 生成规则** | [[wiki/dms90/概念/文档ID生命周期]] + [[wiki/dms90/概念/shardKey分表机制]] | 🔴 高 |
| **存储后端切换** | [[wiki/dms90/概念/双存储适配]] + [[wiki/dms90/概念/多租户隔离]] | 🔴 高 |
| **限流配置** | [[wiki/dms90/概念/限流机制]] + [[wiki/dms90/实体/Sentinel]] | 🟡 中 |
| **缓存策略** | [[wiki/dms90/概念/缓存架构]] + [[wiki/dms90/实体/Redis]] | 🟡 中 |
| **API 签名** | [[wiki/dms90/来源/2026-04-20 代码分析-dms-web]] | 🟢 低 |

---

## 🚀 最佳实践

### 1. 问题解决后

- ✅ 将新问题摄入知识库（如果是新问题）
- ✅ 在相关概念页中添加入链（方便后续查找）
- ✅ 更新 [[wiki/dms90/概念/现场问题快速检索指南]]

### 2. 重构前

- ✅ 读代码分析 → 理解设计意图
- ✅ 查现场问题 → 避免重蹈覆辙
- ✅ 用图谱视图 → 看关联影响

### 3. 代码评审时

- ✅ 对照概念页 → 检查是否符合设计
- ✅ 查现场问题 → 检查是否引入已知问题
- ✅ 看实体页 → 确认使用正确

### 4. 新人培训

- ✅ 按"新人上手路径"逐步学习
- ✅ 每学完一个概念 → 跑通一个现场问题
- ✅ 用图谱视图 → 建立知识关联

---

## 相关页面

- [[wiki/dms90/概念/现场问题快速检索指南]] — 现场问题速查表
- [[wiki/dms90/概念/Obsidian图谱配置说明]] — 图谱视图优化配置
- [[wiki/dms90/概念/DMS MongoDB常用查询]] — MongoDB 查询语句参考
- [[wiki/index]] — 知识库全局索引
- [[wiki/log]] — 操作日志

---

*最后更新：2026-04-20*
