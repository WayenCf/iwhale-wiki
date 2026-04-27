---
type: source
tags: [代码分析, dms-service-impl, 业务层]
sources: []
created: 2026-04-20
updated: 2026-04-20
---


# 代码分析：dms-service-impl

> DMS 核心服务实现层精读，包含文档生成、批量处理、加密签名、定时扫描、模版发布、PDF 签名等全部业务逻辑。

## 模块职责

- **基础包**：`com.iwhalecloud.ftf.dms.service`
- **核心类数量**：110 个 Java 文件
- **关键依赖**：[[wiki/dms90/实体/JasperReports|JasperReports]]（文档渲染）、[[wiki/dms90/实体/Guava Cache|Guava Cache]]（缓存）、PDFBox + BouncyCastle（[[wiki/dms90/概念/PDF签名|PDF 签名]]）、ZMQ（消息队列）

### 包结构

| 包 | 文件数 | 职责 |
|---|---|---|
| `config/` | 9 | 配置属性绑定（批量、切割、加密、线程池、MQ、卷容量等） |
| `impl/` | 19 | Service 接口实现类（核心业务逻辑） |
| `task/` | 21 | 异步任务（文档生成、上传、下载、批量、MQ 消息发送） |
| `encrypt/` | 2 | 文档加密/解密与 Token 令牌 |
| `pdf/` | 5 | [[wiki/dms90/概念/PDF签名|PDF 签名]]、位置提取、工具类 |
| `cutting/` | 10 | 大 XML 文件切割分段 |
| `scanner/` | 4 | 定时文件扫描器 |
| `executor/` | 2 | 自定义线程池执行器 |

## 核心类

### Service 实现

- **DocumentServiceImpl**：[[wiki/dms90/概念/双存储适配|双存储模式]]（MongoDB 优先 → Spool 降级），多租户切换，DFS 模式
- **ResourceServiceImpl**：模版资源管理 + 发布逻辑，MD5 去重，版本号递增（1.2.3 → 1.2.4）
- **ReleaseServiceImpl**：双重检查锁缓存（synchronized + [[wiki/dms90/实体/Guava Cache|LocalCache]]）
- **BatchDocumentServiceImpl**：[[wiki/dms90/概念/批量账单生成流程|批量账单生成]]核心，继承 AbstractBatchService
- **CertificateServiceImpl**：从 ZIP 导入证书包（.crt + .pwd + 签名图片）
- **TaskRecordServiceImpl**：AtomicInteger 线程安全计数，每 20 个任务持久化一次

### PDF 签名

- **BasePdfSigner**：实现 PDFBox SignatureInterface，BouncyCastle CMS SHA256WithRSA
- **PdfSigner**：检查 DocMDP 权限 → 设置签名属性 → 可见签名 → 增量保存
- **PdfUtil**：提取 "DMS-SIGNATURE" 关键字位置 → 签名

### 加密

- **DocEncryptor**：PKCS12 KeyStore + X509 证书 + SHA256WithRSA 签名/验证，带 [[wiki/dms90/实体/Guava Cache|LocalCache]] 缓存
- **Token**：RSA-2048 加密 JSON → 拼接密文+证书ID+长度 → Base64 编码

## 关键发现

1. **[[wiki/dms90/概念/批量账单生成流程|批量账单生成完整流程]]**：扫描 → XML 解析 → 任务拆分 → 并发生成 → 上传 → 标签 → 索引
2. **[[wiki/dms90/概念/PDF签名|PDF 签名链]]**：BouncyCastle CMS → PDFBox 增量保存 → DocMDP 权限控制 → 可见签名
3. **双存储贯穿**：DocumentServiceImpl/FileServiceImpl 的 CUD 操作全部 MongoDB 优先 → Spool 降级
4. **多线程模型**：docMakingExecutor（文档生成）+ repositoryExecutor（落盘）+ DmsExecutor（CallerRunsPolicy）
5. **XML 切割**：DivisibleXml 实现 Iterable<Segment>，支持全局段/头部/明细/尾部四种段类型，递归嵌套规则

## 设计模式

- **策略模式**：AbstractBatchService 定义抽象方法，BatchDocumentServiceImpl/BatchFileServiceImpl 分别实现
- **工厂模式**：DocMakeTaskFactory 按配置创建不同参数的 DocMakeTask
- **模板方法**：BaseEntityTask 定义 onSuccess/onFailure 模板
- **迭代器模式**：DivisibleXml 实现 Iterable<Segment>
- **观察者/事件**：MQ 消息驱动文档生成（DocMakeRequestHandler）
- **条件化装配**：大量 @ConditionalOnProperty 控制功能开关

## 关联概念

- [[wiki/dms90/概念/批量账单生成流程|批量账单生成流程]]
- [[wiki/dms90/概念/PDF签名|PDF 签名]]
- [[wiki/dms90/概念/双存储适配|双存储适配]]
- [[wiki/dms90/概念/缓存架构|缓存架构]]
- [[wiki/dms90/概念/实体继承体系|实体继承体系]]
- [[wiki/dms90/概念/shardKey分表机制|shardKey 分表机制]]
- [[wiki/dms90/概念/多租户隔离|多租户隔离]]
