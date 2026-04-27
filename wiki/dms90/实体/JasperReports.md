---
title: JasperReports
type: entity
created: 2026-04-19
updated: 2026-04-20
tags: [组件, 报表引擎, 模版渲染, 现场问题, 代码分析]
sources: []
---

# JasperReports

> DMS 使用的文档/账单模板渲染引擎，负责将模板 + XML 数据渲染为 PDF/HTML/CSV/TXT 等格式输出。

## 在 DMS 中的角色

| 方面 | 详情 |
|------|------|
| 版本 | 6.10.0 |
| Maven 模块 | `dms-jasperreport-ext`（渲染扩展）<br>`dms-docmaker`（模板生成引擎）<br>`dms-designer-plugins`（设计器插件） |
| 用途 | 账单/文档生成（`DocMakeTask` 调用渲染） |

## 文档生成流程

```
模板 (template URI) + XML 数据 → JasperReports 渲染 → PDF/HTML/CSV/TXT
```

### 模板版本管理

- 模板有三个状态区：`draft` / `prerelease` / `release`
- 支持按历史时间点回溯版本（`date` 参数）
- `EDIT_HISTORY_TEMPLATE_ENABLE=true` 时启用历史版本查找

### 输出格式

- **PDF**：默认格式，支持加密（`pdfPassword`）和签名（`sigPosition`）
- **HTML**：网页预览
- **CSV**：表格数据
- **TXT**：纯文本，可配置字体（`textConfig`）

## 大文件处理

- `xmlPath` 传入时 → `HeavyDocMakeTask`（强制异步）
- 支持 `cuttingRules` 大账单切分
- 支持 `jrprintPath` 中间文件路径

## 在现场问题中的角色

### XML 与模版不匹配

- [[wiki/dms90/来源/2024-03-21 GAIA项目账单生成异常]]：XML 和模版不匹配导致渲染后账单为空白（DMS 无报错日志）

### 字段超长导致死循环

- [[wiki/dms90/来源/2024-03-12 卢旺达文件上传下载失败]]：`CUST_BILL_ADDR` 字段超长导致 `JRBaseFiller.fillf` 执行 waiting 死循环
- [[wiki/dms90/来源/2024-08-23 MPT批量XML上传接口Running]]：同样的字段超长死循环问题

### 文字宽度计算不一致

- [[wiki/dms90/来源/2024-08-14 MPT模版设计器预览与接口生成不一致]]：后台计算文字宽度与 PDF 实际宽度不一致导致截断
- 解决配置：`net.sf.jasperreports.export.pdf.force.linebreak.policy=true`

### 离线账单验证

- [[wiki/dms90/来源/2026-03-12 阿富汗ETA离线生成账单失败]]：通过本地模拟验证离线生成功能

## 常见故障模式

1. **XML 与模版不匹配** → 空白账单（无报错日志）
2. **字段超长** → `JRBaseFiller.fillf` 死循环 → `task_record` 卡在 running
3. **文字宽度计算偏差** → PDF 文本截断

## 关键配置

| 配置项 | 说明 |
|--------|------|
| `net.sf.jasperreports.export.pdf.force.linebreak.policy` | 强制使用 PDF 行宽计算，避免截断 |

## 代码级细节（来自代码分析）

### 文档生成集成

- **DocGenerator.generateDoc()**（docmaker 模块，service-impl 调用）：
  - 数据源：XML 数据流（xmlDataStream）或 JDBC 数据源（dataSourceAdapter）
  - 模版：从 Release 的 Resource Map 中获取 .jasper 编译后文件
  - 输出格式：FileType.pdf / 其他
  - 参数：entity.getParams() + locale + textConfig + pdfPassword
  - CSV 配置：CsvFieldProperties.delimiter

### DocMakerResource

- 服务层实现的 IResource 接口
- 将 Resource Map 包装为 JasperReports 可访问的资源加载器

### 模版类型与配对

- `.jrxml`：JasperReports XML 源文件
- `.jasper`：JasperReports 编译后文件
- 两者总是配对存在（addCouple 方法确保关联）

### 版本号管理

- `getNextVersion(version)`：递增最后一段数字（如 1.2.3 → 1.2.4）
- HistoryTemplateServiceImpl.getPublishInfo() 计算 nextVersion（如 1.0 → 1.0.1）

### 新接口 vs 旧接口

| 类 | 返回类型 | 特点 |
|---|---|---|
| DocMakeTask | ByteArrayOutputStream | 旧接口，异步提交 DocUploadTask |
| BatchDocMakeTask | DocMakeTaskResult | 新接口（v2），同步等待上传完成（5 分钟超时），有 TEMPLATE_CACHE |
| HeavyDocMakeTask | ByteArrayOutputStream | 大账单切割生成 |

### dms-web 兼容层

- **Jasperserver**：POST/GET /dms/j_spring_security_check — JasperServer 登录接口代理
- **ResourcesServlet**：/dms/rest_v2/resources/** — REST 接口代理，XML 数据格式（JAXB）
- **RestV2**：/dms/rest_v2/serverInfo — 硬编码版本 6.2.0
- **Services**：/dms/services/repository — SOAP 响应代理

### ExpImpServiceImpl 模版导入导出

- **导出**：查询资源+关联 → 反射遍历 getter 生成 XML → 文件 Base64 编码嵌入 → ZIP 打包
- **导入**：解压 ZIP → 解析 DMS_RESOURCE.xml → 逐资源 create/update（version+1）→ 按版本排序发布

## 关联页面

- [[wiki/dms90/来源/2026-04-19 DMS项目概览|DMS 项目概览]]
- [[wiki/dms90/来源/2026-04-19 DocumentController接口逻辑梳理|DocumentController 接口]]
- [[wiki/dms90/实体/iWhaleCloud|iWhaleCloud]]
- [[wiki/dms90/来源/2024-03-21 GAIA项目账单生成异常]]
- [[wiki/dms90/来源/2024-03-12 卢旺达文件上传下载失败]]
- [[wiki/dms90/来源/2024-08-23 MPT批量XML上传接口Running]]
- [[wiki/dms90/来源/2024-08-14 MPT模版设计器预览与接口生成不一致]]
- [[wiki/dms90/来源/2026-03-12 阿富汗ETA离线生成账单失败]]

## 关联概念

- [[wiki/dms90/概念/PDF签名|PDF 签名]]
