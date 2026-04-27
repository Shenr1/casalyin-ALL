# 产品表单 UI 优化 - 发现与决策

## 任务背景
产品详情页表单太长（695行），用户添加时有心理压力。需要从 UI 方面优化，但字段功能不改。

## 已完成工作
1. 产品标签交互已优化为 Select 下拉框（3列布局，必选的排在前面）
2. 已修改文件：ProductForm.tsx、i18n 三语言文件

## 待完成工作

### 1. 修正必填字段校验规则
**当前状态：** 
- SKU、产品名称、品牌、价格、联系人姓名、联系电话都是必填

**目标状态：**
- 必填：产品名称、品牌、有效成分(compositionIds)、必填的标签
- 选填：SKU、关联店铺、简述、详细描述、价格、联系方式、产品图片、产品文档

**修改位置：**
- ProductForm.tsx 第 177、544、593、601 行（移除 required: true）
- ProductForm.tsx 第 493 行（添加 compositionIds 必填校验）

### 2. 将产品文档功能从 ProductEditor 移到 ProductForm
**参考实现：** ProductEditor.tsx 第 1335-1388 行

**需要添加的 Props：**
```typescript
documentList?: ProductDocument[]
onDocumentListChange?: (list: ProductDocument[]) => void
onDocumentUpload?: (file: File) => Promise<string>
```

**实现要点：**
- 使用 Upload 组件，限制文件类型（PDF、Word、Excel）
- 文件大小限制 5MB
- 支持文档标题编辑
- 支持删除文档

### 3. 实现 Collapse 折叠面板

**区块划分：**

**默认展开（必填区域）：**
- 基础信息：产品名称、品牌
- 标签与成分：标签信息、有效成分

**默认折叠（选填区域）：**
- 补充信息：SKU、关联店铺、简述
- 详细描述：富文本编辑器
- 价格与联系：价格信息、联系方式
- 图片与文档：产品图片、产品文档
- 仓储物流（仅 Admin）：可购城市、库存、Discourse Tag

### 4. i18n 新增 Key

需要在三个语言文件中添加：
```json
"productForm": {
  "sectionBasicInfo": "基础信息",
  "sectionTagsAndCompositions": "标签与成分",
  "sectionAdditionalInfo": "补充信息",
  "sectionDescription": "详细描述",
  "sectionPriceAndContact": "价格与联系",
  "sectionMediaAndDocs": "图片与文档",
  "sectionWarehouseLogistics": "仓储物流",
  "compositionRequired": "请至少选择一个有效成分",
  "documentTitle": "产品文档",
  "uploadDocument": "上传文档",
  "uploadTypeError": "仅支持 PDF、Word、Excel 格式",
  "uploadSizeError": "文件大小不能超过 5MB",
  "uploadSuccess": "上传成功",
  "uploadFailed": "上传失败",
  "noDocuments": "暂无文档",
  "documentTitlePlaceholder": "文档标题"
}
```

## 实施策略
由于 ProductForm.tsx 文件较大（695行），采用分段修改策略，每次修改不超过50行。

## 风险点
1. 产品文档功能涉及文件上传，需要确保 API 调用正确
2. Collapse 组件可能影响表单校验触发时机
3. 必填字段变更可能影响现有数据的兼容性
