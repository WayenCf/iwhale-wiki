---
type: source
tags: [现场问题, 字体支持, Tahoma字体, 模版设计, 利比亚项目]
sources: [20240314-利比亚项目需求DMS支持Tahoma字体.md]
created: 2024-03-14
updated: 2026-04-20
---


# 利比亚项目DMS支持Tahoma字体

## 需求背景

现网提供的模版中有 Tahoma 字体，配置组发现当前 [[wiki/dms90/来源/2026-04-19 DMS项目概览|DMS]] 不支持这种字体。

## 业务需求

1. [[wiki/dms90/来源/2026-04-19 DMS项目概览|DMS]] 支持导入 `tahoma.ttf`（常规）、`tahomabd.ttf`（粗体）字体文件
2. [[wiki/dms90/来源/2026-04-19 DMS项目概览|DMS]] 需要支持导入其他字体文件
3. 配置组配置合同等模版可以使用 Tahoma 等字体
4. 业务侧调用模版展示正确

## 修改方案

### 代码修改

1. 修改 `dms-fonts` 模块 `com.iwhalecloud.ftf.dms.fonts` 目录下新增 `tahoma` 目录
2. 目录下增加两个字体文件：
   - `tahoma.ttf`（常规：normal）
   - `tahomabd.ttf`（粗体：bold）
3. `com.iwhalecloud.ftf.dms.fonts` 目录下 `fonts.xml` 增加 `tahomabd` 字体声明
4. 构建 `dms-fonts` jar 包，替换设计器中原有 jar 文件
5. [[wiki/dms90/来源/2026-04-19 DMS项目概览|DMS]] 服务端正常通过版本发布

### 验证要点

1. 验证可以导入 `tahoma`、`tahomabd` 字体
2. 验证配置合同模版支持 `tahoma`、`tahomabd` 字体
3. 验证模版配置好后，业务侧调用模版可以正常展示

## 相关实体

- 利比亚项目
- [[wiki/dms90/来源/2026-04-19 DMS项目概览|DMS]]
- 字体
- 模版设计
