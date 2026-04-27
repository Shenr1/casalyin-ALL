# 第四期：产品表单布局调整 - 2个折叠面板

## 任务背景

用户希望简化产品表单布局，将必填项和选填项分开，降低用户上传产品的压力。

**前置任务（已完成）：**
- ✅ 第一期：产品列表和表单显示问题修复（Commit: f3ea3a9）
- ✅ 第二期：前端必填项规则调整（Commit: 12882f3）
- ✅ 第三期：后端验证逻辑同步修改（Commit: fb371ba）

## 任务目标

将产品表单重新组织为 **2个折叠面板**：
1. **基础配置**（必填项）- 默认展开
2. **高级配置**（选填项）- 默认折叠

## 布局方案

### 1. 基础配置（Basic Configuration）- 默认展开

**包含字段（必填项）：**
- 产品名称（name）- 必填
- 品牌（brandId）- 必填
- 有效成分（compositionIds）- 必填
- 必选标签分类（tagIds）- 必填

### 2. 高级配置（Advanced Configuration）- 默认折叠

**包含字段（选填项）：**
- SKU（sku）- 非必填
- 关联店铺（stores）- 非必填
- 产品简介（info）- 非必填
- 价格列表（prices）- 非必填
- 联系人列表（contacts）- 非必填
- 非必选标签分类 - 非必填
- 产品描述（description）- 非必填
- 产品图片（pictures）- 非必填
- 产品文档（documents）- 非必填
- 其他字段（discourseTag、availableCities、stockQuantity 等）

## 修改文件

1. `src/components/ProductForm.tsx` - 主要修改
2. `src/locales/zh-CN.json` - 添加 i18n 文案
3. `src/locales/en.json` - 添加 i18n 文案
4. `src/locales/es-ES.json` - 添加 i18n 文案

## 具体实施步骤

### 步骤 1：添加 i18n 文案

**zh-CN.json**：
```json
{
  "productForm": {
    "basicConfig": "基础配置",
    "advancedConfig": "高级配置"
  }
}
```

**en.json**：
```json
{
  "productForm": {
    "basicConfig": "Basic Configuration",
    "advancedConfig": "Advanced Configuration"
  }
}
```

**es-ES.json**：
```json
{
  "productForm": {
    "basicConfig": "Configuración Básica",
    "advancedConfig": "Configuración Avanzada"
  }
}
```

### 步骤 2：修改 ProductForm.tsx 的折叠面板结构

**修改 Collapse 组件的 defaultActiveKey**：
```typescript
// 修改前
<Collapse defaultActiveKey={['basic', 'tags']} ...>

// 修改后
<Collapse defaultActiveKey={['basic']} ...>
```

**重新组织折叠面板**：

```typescript
<Collapse
  defaultActiveKey={['basic']}
  style={{ marginBottom: 16 }}
  bordered={false}
>
  {/* ========== 基础配置 - 默认展开 ========== */}
  <Collapse.Panel header={t('productForm.basicConfig')} key="basic">
    
    {/* 1. 产品名称 - 必填 */}
    <Form.Item
      name="name"
      label={t('productForm.productName')}
      required={!readonly}
      rules={readonly ? [] : [{ required: true, message: t('productForm.productNameRequired') }]}
    >
      <Input placeholder={t('productForm.productNamePlaceholder')} disabled={readonly} />
    </Form.Item>

    {/* 2. 品牌 - 必填 */}
    {/* 将品牌选择相关的代码移动到这里 */}
    {/* 包括 companyBrand 的判断逻辑和 BrandSelect 组件 */}

    {/* 3. 有效成分 - 必填 */}
    <Form.Item
      name="compositionIds"
      label={t('productForm.compositions')}
      required={!readonly}
      rules={readonly ? [] : [{ required: true, message: t('productForm.compositionRequired') }]}
    >
      <CompositionSelect ... />
    </Form.Item>

    {/* 4. 必选标签分类 - 必填 */}
    {/* 只渲染 isRequired === true 的标签分类 */}
    {tagCategories
      .filter(category => category.isRequired)
      .map(category => (
        // 标签分类渲染逻辑
      ))
    }

  </Collapse.Panel>

  {/* ========== 高级配置 - 默认折叠 ========== */}
  <Collapse.Panel header={t('productForm.advancedConfig')} key="advanced">
    
    {/* 1. SKU - 非必填 */}
    <Form.Item name="sku" label={t('productForm.sku')}>
      <Input placeholder={t('productForm.skuPlaceholder')} disabled={readonly} />
    </Form.Item>

    {/* 2. 关联店铺 - 非必填 */}
    {showStores && (
      <Form.Item name="stores" label={t('productForm.stores')}>
        <Select ... />
      </Form.Item>
    )}

    {/* 3. 产品简介 - 非必填 */}
    <Form.Item name="info" label={t('productForm.summary')}>
      <Input.TextArea ... />
    </Form.Item>

    {/* 4. 价格列表 - 非必填 */}
    <Form.Item label={t('productForm.pricesLabel')}>
      <Form.List name="prices">
        {/* 价格列表渲染逻辑 */}
      </Form.List>
    </Form.Item>

    {/* 5. 联系人列表 - 非必填 */}
    <Form.Item label={t('productForm.contactsLabel')}>
      <Form.List name="contacts">
        {/* 联系人列表渲染逻辑 */}
      </Form.List>
    </Form.Item>

    {/* 6. 非必选标签分类 - 非必填 */}
    {/* 只渲染 isRequired === false 的标签分类 */}
    {tagCategories
      .filter(category => !category.isRequired)
      .map(category => (
        // 标签分类渲染逻辑
      ))
    }

    {/* 7. 产品描述 - 非必填 */}
    <Form.Item name="description" label={t('productForm.descriptionLabel')}>
      <RichTextEditor ... />
    </Form.Item>

    {/* 8. 产品图片 - 非必填 */}
    <Form.Item name="pictures" label={t('productForm.picturesLabel')}>
      <ImageUploadField ... />
    </Form.Item>

    {/* 9. 其他字段 */}
    {/* documents、usages、discourseTag、availableCities、stockQuantity 等 */}

  </Collapse.Panel>
</Collapse>
```

### 步骤 3：关键注意事项

1. **标签分类的分离**：
   - 必选标签分类（isRequired === true）放在"基础配置"
   - 非必选标签分类（isRequired === false）放在"高级配置"

2. **保持现有逻辑**：
   - 不要修改字段的验证规则（第二期已完成）
   - 不要修改字段的渲染逻辑
   - 只是移动字段的位置

3. **品牌字段的特殊处理**：
   - 需要保留 companyBrand 的判断逻辑
   - 需要保留 BrandSelect 组件的所有逻辑
   - 需要保留隐藏字段（customBrandName、brandDisplayName）

4. **只读模式的处理**：
   - 保持 readonly 参数的所有逻辑
   - 不要修改只读模式下的显示方式

## 测试建议

1. **创建产品测试**：
   - 只填写必填字段（产品名称、品牌、有效成分、必选标签）
   - 确认可以成功创建产品
   - 确认高级配置默认折叠

2. **编辑产品测试**：
   - 打开已有产品
   - 确认必填字段在"基础配置"中
   - 确认选填字段在"高级配置"中

3. **只读模式测试**：
   - 查看产品详情
   - 确认所有字段正确显示

4. **标签分类测试**：
   - 确认必选标签分类在"基础配置"中
   - 确认非必选标签分类在"高级配置"中

## 预计时间

20-30 分钟

## 提交信息

```
refactor: 优化产品表单布局 - 分离必填和选填项

将产品表单重新组织为2个折叠面板：
- 基础配置（必填项）- 默认展开
- 高级配置（选填项）- 默认折叠

目标：降低用户上传产品的压力，只需填写必填信息即可快速创建产品

修改文件：
- ProductForm.tsx - 重新组织折叠面板布局
- zh-CN.json, en.json, es-ES.json - 添加 i18n 文案

Co-Authored-By: Claude Sonnet 4.6 <noreply@anthropic.com>
```
