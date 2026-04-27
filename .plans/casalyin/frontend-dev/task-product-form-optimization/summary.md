# 产品表单 UI 优化任务 - 当前状态

## 任务概述
优化产品详情页表单 UI，通过 Collapse 折叠面板减少首屏展示内容，降低用户心理压力。

## 已完成工作
1. ✅ 产品标签交互已优化为 Select 下拉框（3列布局，必选的排在前面）
2. ✅ 已修改 ProductForm.tsx Props 接口，添加文档相关参数

## 待完成工作

### 1. 修正必填字段校验规则
**需要修改的位置：**
- 第 177 行：移除 SKU 的 required: true
- 第 493 行：添加 compositionIds 必填校验
- 第 544 行：移除价格的 required: true
- 第 593 行：移除联系人姓名的 required: true
- 第 601 行：移除联系电话的 required: true

### 2. 添加产品文档功能
参考 ProductEditor.tsx 第 1335-1388 行，需要在 ProductForm 中添加：
- Upload 组件（限制 PDF/Word/Excel，5MB）
- 文档列表展示
- 文档标题编辑
- 删除文档功能

### 3. 实现 Collapse 折叠面板
将表单内容重新组织为折叠面板结构：
- 默认展开：基础信息、标签与成分
- 默认折叠：补充信息、详细描述、价格与联系、图片与文档、仓储物流

### 4. 同步 i18n 三语言
需要添加的 key：
- sectionBasicInfo, sectionTagsAndCompositions, sectionAdditionalInfo
- sectionDescription, sectionPriceAndContact, sectionMediaAndDocs
- sectionWarehouseLogistics, compositionRequired
- documentTitle, uploadDocument, uploadTypeError, uploadSizeError
- uploadSuccess, uploadFailed, noDocuments, documentTitlePlaceholder

### 5. 创建规范文档
在 docs/frontend/ 创建 form-collapse-pattern.md

## 实施建议
由于 ProductForm.tsx 文件较大（695行），建议采用以下策略：
1. 先完成小的修改（必填字段校验）
2. 再添加产品文档功能
3. 最后实现 Collapse 折叠面板（这是最大的改动）
4. 同步 i18n 和创建文档

## 需要 team-lead 确认
1. 是否需要保持向后兼容（现有产品数据可能没有填写有效成分）？
2. 产品文档功能的 API 调用方式（onDocumentUpload 回调的具体实现）
3. Collapse 面板的默认展开/折叠状态是否符合预期？

## 预计工作量
- 代码修改：2-3小时
- i18n 同步：30分钟
- 文档编写：30分钟
- 测试验证：1小时
