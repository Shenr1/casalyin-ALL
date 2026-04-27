# 产品表单 UI 优化任务

> 状态: IN_PROGRESS
> 负责人: frontend-dev
> 创建时间: 2026-04-24

## 任务背景

产品详情页表单太长（710行），用户添加时会有心理压力。需要从 UI 方面优化，但字段功能不改。

## 已完成的工作

1. ✅ 产品标签交互优化为 Select 下拉框（3列布局，必选的排在前面）
2. ✅ ProductForm.tsx 已添加产品文档相关接口定义（ProductDocument、documentList、onDocumentListChange、onDocumentUpload）

## 待完成任务

### 第一步：修正必填字段校验规则

**必填字段（只有这些）：**
- 产品名称（name）- 第 202 行 ✓ 已经是必填
- 品牌（brandId）- 第 218 行 ✓ 已经是必填
- 有效成分（compositionIds）- 第 556 行 ❌ 需要添加必填校验
- 必填的标签（tagIds）- 第 415 行 ✓ 已经有必填校验

**需要移除必填的字段：**
- SKU（sku）- 第 193 行，移除 `required: true`
- 价格（prices）- 第 544 行，移除 `required: true`
- 联系方式（contacts）- 第 593 行和 601 行，移除 `required: true`

**修改清单：**
```typescript
// 1. 第 193 行 - SKU 改为选填
rules={readonly ? [] : [{ required: false }]} // 或直接移除 rules

// 2. 第 556 行 - 有效成分改为必填
<Form.Item 
  name="compositionIds" 
  label={t('productForm.compositionSelectLabel')}
  rules={readonly ? [] : [{ required: true, message: t('productForm.compositionRequired') }]}
>

// 3. 第 544 行 - 价格改为选填（移除 required）
name={[name, 'price']}
rules={readonly ? [] : []} // 移除 required

// 4. 第 593、601 行 - 联系方式改为选填（移除 required）
name={[name, 'name']}
rules={readonly ? [] : []} // 移除 required

name={[name, 'phone']}
rules={readonly ? [] : []} // 移除 required
```

### 第二步：实现产品文档功能

将 ProductEditor.tsx 第 1335-1388 行的产品文档上传功能集成到 ProductForm 中。

**需要添加的内容：**
1. 文档上传 Upload 组件
2. 文档列表展示 List 组件
3. 文档标题编辑功能
4. 文档删除功能

**位置：** 在产品图片（第 631-649 行）之后添加

**代码结构：**
```typescript
{/* 产品文档 */}
{!readonly && onDocumentUpload && (
  <Form.Item label={t('productForm.documentLabel')}>
    <Space direction="vertical" style={{ width: '100%' }} size={12}>
      <Upload
        multiple={false}
        showUploadList={false}
        beforeUpload={(file) => {
          // 文件类型和大小校验
        }}
        customRequest={async ({ file, onSuccess, onError }) => {
          // 调用 onDocumentUpload 上传文件
        }}
      >
        <Button icon={<PlusOutlined />}>{t('productForm.uploadDocument')}</Button>
      </Upload>
      
      {documentList.length > 0 && (
        <List
          bordered
          dataSource={documentList}
          renderItem={(item, index) => (
            // 文档列表项
          )}
        />
      )}
    </Space>
  </Form.Item>
)}
```

### 第三步：实现 Collapse 折叠面板

使用 Ant Design 的 Collapse 组件重构表单结构。

**折叠面板分组：**

1. **基础信息**（默认展开，key: 'basic'）
   - 产品名称、品牌

2. **标签与成分**（默认展开，key: 'tags'）
   - 标签信息、有效成分

3. **补充信息**（默认折叠，key: 'additional'）
   - SKU、关联店铺、简述

4. **详细描述**（默认折叠，key: 'description'）
   - 富文本编辑器

5. **价格与联系**（默认折叠，key: 'pricing'）
   - 价格信息、联系方式

6. **图片与文档**（默认折叠，key: 'media'）
   - 产品图片、产品文档

7. **仓储物流**（默认折叠，key: 'logistics'，仅 Admin）
   - 可购城市、库存、Discourse Tag

**代码结构：**
```typescript
<Collapse 
  defaultActiveKey={['basic', 'tags']} 
  style={{ marginBottom: 16 }}
>
  <Collapse.Panel header={t('productForm.basicInfo')} key="basic">
    {/* 产品名称、品牌 */}
  </Collapse.Panel>
  
  <Collapse.Panel header={t('productForm.tagsAndCompositions')} key="tags">
    {/* 标签信息、有效成分 */}
  </Collapse.Panel>
  
  {/* 其他折叠面板... */}
</Collapse>
```

### 第四步：同步 i18n 三语言

**需要添加的翻译 key：**

```json
// zh-CN.json
"productForm": {
  "compositionRequired": "请选择有效成分",
  "documentLabel": "产品文档",
  "uploadDocument": "上传文档",
  "documentTitlePlaceholder": "文档标题",
  "noDocuments": "暂无文档",
  "uploadTypeError": "仅支持 PDF、Word、Excel 格式",
  "uploadSizeError": "文件大小不能超过 5MB",
  "uploadSuccess": "上传成功",
  "uploadFailed": "上传失败",
  "basicInfo": "基础信息",
  "tagsAndCompositions": "标签与成分",
  "additionalInfo": "补充信息",
  "detailedDescription": "详细描述",
  "pricingAndContact": "价格与联系",
  "mediaFiles": "图片与文档",
  "logistics": "仓储与物流"
}

// en.json
"productForm": {
  "compositionRequired": "Please select compositions",
  "documentLabel": "Product Documents",
  "uploadDocument": "Upload Document",
  "documentTitlePlaceholder": "Document title",
  "noDocuments": "No documents",
  "uploadTypeError": "Only PDF, Word, Excel formats are supported",
  "uploadSizeError": "File size cannot exceed 5MB",
  "uploadSuccess": "Upload successful",
  "uploadFailed": "Upload failed",
  "basicInfo": "Basic Information",
  "tagsAndCompositions": "Tags & Compositions",
  "additionalInfo": "Additional Information",
  "detailedDescription": "Detailed Description",
  "pricingAndContact": "Pricing & Contact",
  "mediaFiles": "Images & Documents",
  "logistics": "Warehouse & Logistics"
}

// es-ES.json
"productForm": {
  "compositionRequired": "Por favor seleccione composiciones",
  "documentLabel": "Documentos del Producto",
  "uploadDocument": "Subir Documento",
  "documentTitlePlaceholder": "Título del documento",
  "noDocuments": "Sin documentos",
  "uploadTypeError": "Solo se admiten formatos PDF, Word, Excel",
  "uploadSizeError": "El tamaño del archivo no puede exceder 5MB",
  "uploadSuccess": "Subida exitosa",
  "uploadFailed": "Subida fallida",
  "basicInfo": "Información Básica",
  "tagsAndCompositions": "Etiquetas y Composiciones",
  "additionalInfo": "Información Adicional",
  "detailedDescription": "Descripción Detallada",
  "pricingAndContact": "Precios y Contacto",
  "mediaFiles": "Imágenes y Documentos",
  "logistics": "Almacén y Logística"
}
```

### 第五步：更新 ProductEditor.tsx

移除 ProductEditor 中重复的文档上传代码（第 1335-1388 行），改为通过 props 传递给 ProductForm。

**需要修改的地方：**
1. 将 documentList 状态和相关函数通过 props 传递给 ProductForm
2. 移除 ProductEditor 中的文档上传 UI 代码
3. 确保所有模式（create/draft/detail）都能正常工作

### 第六步：创建规范文档

创建 `docs/frontend/form-collapse-pattern.md`，记录折叠面板设计模式。

**文档内容：**
- 折叠面板的使用场景
- 分组原则（必填/选填、相关性）
- 默认展开/折叠规则
- 代码示例
- 应用到其他表单的指南

## 验收标准

1. ✅ 必填字段校验正确（只有产品名称、品牌、有效成分、必填标签是必填的）
2. ✅ 产品文档功能在所有模式下都可见
3. ✅ Collapse 折叠面板实现，必填区域默认展开，选填区域默认折叠
4. ✅ 用户首次看到的表单内容大幅减少
5. ✅ i18n 三语言同步
6. ✅ 规范文档完整
7. ✅ ProductEditor 中的重复代码已移除

## 注意事项

- 项目已经使用 Ant Design，遵循现有的样式规范
- 不要改变字段的功能，只优化 UI 展示
- 保持代码风格一致
- 每完成一步都要测试验证
