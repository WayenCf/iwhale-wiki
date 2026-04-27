---
title: PDF签名
type: concept
created: 2026-04-20
updated: 2026-04-20
tags: [security, pdf, signature]
sources: []
---

# PDF 签名

> DMS 使用 BouncyCastle CMS 和 PDFBox 实现 PDF 数字签名，支持 SHA256WithRSA 算法、可见签名、DocMDP 权限控制、关键字定位签名等高级功能。

## 签名链路

```
DocEncryptor（证书管理）
    → BasePdfSigner（PDFBox SignatureInterface）
    → PdfSigner（签名执行）
    → PdfUtil（关键字定位）
```

## 核心组件

### 1. DocEncryptor — 证书管理

| 方法 | 说明 |
|------|------|
| getKeyStore(cak) | 加载 PKCS12 KeyStore，带 LocalCache 缓存 |
| getPrivateKey(cak) | 从 KeyStore 提取私钥（别名 "1"） |
| getPublicKey(cak) | 从 KeyStore 提取公钥 |
| getCertificate(cak) | 从 KeyStore 提取证书 |
| sign(data, id) | 使用指定证书对数据签名 |
| verify(data, sign, crt) | 使用给定证书验证签名 |

**加密算法**：PKCS12 KeyStore + X509 证书 + SHA256WithRSA

### 2. BasePdfSigner — PDFBox 签名接口

- **实现**：PDFBox 的 SignatureInterface
- **构造时**：从 PKCS12 KeyStore 提取私钥和证书链
- **sign()**：
  1. 使用 BouncyCastle CMS 签名
  2. SHA256WithRSA 算法
  3. 生成 PKCS#7 签名数据

### 3. PdfSigner — 签名执行

**signPDF(inputStream, outputStream, position)** 流程：

1. 加载 PDF 文档（PDDocument.load）
2. 检查 DocMDP 权限（getMDPPermission）
3. 设置签名属性：
   - Filter: ADBE_PPKLITE
   - SubFilter: ADBE_PKCS7_DETACHED
4. 如有签名图片和位置，创建可见签名（PDVisibleSigProperties）
5. doc.addSignature() + doc.saveIncremental() 增量保存

### 4. PdfUtil — 关键字定位

**sign(cakId, inputStream, outputStream)** 流程：

1. 获取证书（DocEncryptor）
2. 创建 PdfSigner
3. 提取 "DMS-SIGNATURE" 关键字位置（extractPositions）
4. 执行签名

**extractPositions(inputStream, keywords)**：

- 使用 TextPositionAwareStripper
- 提取关键字在 PDF 中的坐标（x, y, width, height）
- 支持多个关键字匹配

### 5. SigUtil — DocMDP 管理

| 方法 | 说明 |
|------|------|
| getMDPPermission(doc) | 获取 DocMDP 权限 |
| setMDPPermission(doc, signature, accessPermissions) | 设置 DocMDP 变换参数 |
| checkCertificateUsage(x509Certificate) | 校验证书用途（digitalSignature / nonRepudiation / emailProtection / codeSigning） |

## 签名流程图

```
上传证书（CertificateServiceImpl.importCrt）
    → 解析 ZIP 包（.crt + .pwd + 签名图片）
    → 存储到 CertificateAttachDao

生成 PDF 时签名（DocUploadTask.doUpload）
    → VolumeService.preCheck() 容量预检
    → PdfUtil.sign(cakId, inputStream, outputStream)
        → DocEncryptor.getCertificate(cakId)
        → PdfSigner.signPDF()
            → BasePdfSigner.sign() (BouncyCastle CMS)
            → PDVisibleSigProperties (可见签名)
            → doc.addSignature()
            → doc.saveIncremental()
    → DocumentAttachDao.uploadFile()
```

## 签名属性配置

| 属性 | 值 | 说明 |
|------|---|------|
| Filter | ADBE_PPKLITE | Adobe PKCS#7 |
| SubFilter | ADBE_PKCS7_DETACHED | 分离式签名 |
| SignDate | 当前时间 | 签名时间 |
| Location | DMS | 签名位置 |

## 可见签名

### PDVisibleSigProperties

- **签名图片**：从证书附件加载签名图片
- **位置**：通过关键字定位（extractPositions）
- **大小**：自适应 PDF 页面大小

### 关键字定位

- **关键字**：默认 "DMS-SIGNATURE"
- **提取器**：TextPositionAwareStripper
- **坐标**：(x, y, width, height)

## DocMDP 权限控制

### 权限级别

| 级别 | 说明 |
|------|------|
| 1 | 禁止任何修改 |
| 2 | 允许表单填写和签名 |
| 3 | 允许表单填写、签名和注释 |

### 设置方法

```java
SigUtil.setMDPPermission(doc, signature, accessPermissions);
```

## 证书校验

### 证书用途校验

checkCertificateUsage() 校验证书是否支持以下用途：

- digitalSignature（数字签名）
- nonRepudiation（不可抵赖）
- emailProtection（邮件保护）
- codeSigning（代码签名）

## Token 令牌机制

### Token 结构

| 字段 | 类型 | 说明 |
|------|------|------|
| id | UUID | 令牌 ID |
| user | String | 用户 |
| expire | Date | 过期时间 |
| type | String | 类型 |
| targets | List<String> | 目标列表 |

### 加密流程 toRaw()

1. 将 Token 对象序列化为 JSON
2. 选择有效证书（chooseCak）
3. RSA-2048 加密 JSON
4. 拼接：`密文 + 证书ID + 证书ID长度(2位)`

### 解密流程 fromRaw()

1. 从末尾 2 位取证书 ID 长度
2. 取出证书 ID
3. 取出密文
4. RSA-2048 解密
5. 反序列化为 Token 对象

## 配置属性（EncryptProperties）

| 属性 | 说明 |
|------|------|
| pwdFileExt | 密码文件后缀 |
| crtFileExt | 证书文件后缀 |
| sigImageFileExts | 签名图片后缀数组 |
| validityPeriod | Token 有效期 |

## 关联概念

- [[wiki/dms90/实体/JasperReports|JasperReports]]
- [[wiki/dms90/概念/批量账单生成流程|批量账单生成流程]]

## 关联页面

- [[wiki/dms90/来源/2026-04-20 代码分析-dms-service-impl|代码分析：dms-service-impl]]
