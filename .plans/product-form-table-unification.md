# 产品表单字段统一改造任务

## 📋 任务概述

**目标：** 统一产品详情页中可新增字段的展示样式，提升用户体验和操作效率

**优先级：** P1（高优先级）

**预计工时：** 2-3 天

**负责人：** 待分配

**Review 人：** 待分配

---

## 🎯 改造范围

### 需要改造的字段（使用表格形式）

1. ✅ **联系方式**（phones）
2. ✅ **价格信息**（prices）
3. ✅ **上传文档**（documents）

### 保持当前样式的字段

1. ❌ **标签分类**（tags）- 保持 Collapse + Checkbox 样式
2. ❌ **上传图片**（pictures）- 保持卡片式预览样式

---

## 📐 设计方案

### 参考样式：产品用途管理

**特点：**
- 占据全宽度
- 表格形式展示（使用 `BatchEditTable` 组件）
- 添加按钮在表格下方
- 支持批量操作（批量删除、搜索）
- 视觉清晰，操作流畅

**代码位置：**
```tsx
// 文件：src/pages/ProductEditor.tsx
// 行号：1788-1842

<BatchEditTable<ProductUsageVO>
  columns={buildUsageColumns(detailEditable) as any}
  dataSource={filteredUsageList}
  loading={usageLoading}
  rowKey="usageId"
  rowSelection={...}
  onAddRow={...}
  addRowLabel={t('productDetail.usage.addRow')}
  scroll={{ x: 900 }}
/>
```

---

## 🛠️ 技术实现

### 1. 联系方式（contacts）

**当前实现：** Form.List 组件

**改造后：** BatchEditTable 组件

**表格列配置：**

| 列名 | 字段 | 类型 | 可编辑 | 宽度 |
|------|------|------|--------|------|
| 姓名 | name | Input | ✅ | 200px |
| 电话 | phone | Input | ✅ | 200px |
| 邮箱 | email | Input | ✅ | 200px |
| 操作 | - | Button | - | 100px |

**注意：** 实际实现增加了邮箱字段，提供更完整的联系方式管理。

**代码示例：**

```tsx
const buildPhonesColumns = (editable: boolean): ColumnsType<PhoneInfoDTO> => [
  {
    title: t('productForm.contactName'),
    dataIndex: 'name',
    width: 200,
    render: (text, record, index) => (
      editable ? (
        <Input
          value={text}
          onChange={(e) => handlePhoneChange(index, 'name', e.target.value)}
          placeholder={t('productForm.contactNamePlaceholder')}
        />
      ) : text || '-'
    ),
  },
  {
    title: t('productForm.contactPhone'),
    dataIndex: 'phone',
    width: 200,
    render: (text, record, index) => (
      editable ? (
        <Input
          value={text}
          onChange={(e) => handlePhoneChange(index, 'phone', e.target.value)}
          placeholder={t('productForm.contactPhonePlaceholder')}
        />
      ) : text || '-'
    ),
  },
  {
    title: t('common.actions'),
    width: 100,
    render: (_, record, index) => (
      editable ? (
        <Button
          type="link"
          danger
          onClick={() => handleDeletePhone(index)}
        >
          {t('common.delete')}
        </Button>
      ) : null
    ),
  },
];

// 使用
<BatchEditTable<PhoneInfoDTO>
  columns={buildPhonesColumns(detailEditable)}
  dataSource={phonesList}
  rowKey={(record, index) => `phone-${index}`}
  onAddRow={() => {
    setPhonesList(prev => [...prev, { name: '', phone: '' }]);
  }}
  addRowLabel={t('productForm.addContact')}
/>
```

---

### 2. 价格信息（prices）

**当前实现：** Form.List 组件

**改造后：** BatchEditTable 组件

**表格列配置：**

| 列名 | 字段 | 类型 | 可编辑 | 宽度 |
|------|------|------|--------|------|
| 价格 | price | InputNumber | ✅ | 150px |
| 币种 | currency | Select | ✅ | 120px |
| 单位 | unit | Input | ✅ | 150px |
| 备注 | remark | Input | ✅ | 200px |
| 操作 | - | Button | - | 100px |

**注意：** 实际实现增加了币种和备注字段，提供更完整的价格信息管理。

**原表格列配置（任务文档）：**

| 列名 | 字段 | 类型 | 可编辑 | 宽度 |
|------|------|------|--------|------|
| 价格 | price | InputNumber | ✅ | 200px |
| 单位 | unit | Input | ✅ | 200px |
| 操作 | - | Button | - | 100px |

**代码示例：**

```tsx
const buildPricesColumns = (editable: boolean): ColumnsType<PriceInfoDTO> => [
  {
    title: t('productForm.price'),
    dataIndex: 'price',
    width: 200,
    render: (text, record, index) => (
      editable ? (
        <InputNumber
          value={text}
          onChange={(value) => handlePriceChange(index, 'price', value)}
          placeholder={t('productForm.pricePlaceholder')}
          min={0}
          precision={2}
          style={{ width: '100%' }}
        />
      ) : text ? `$${text}` : '-'
    ),
  },
  {
    title: t('productForm.priceUnit'),
    dataIndex: 'unit',
    width: 200,
    render: (text, record, index) => (
      editable ? (
        <Input
          value={text}
          onChange={(e) => handlePriceChange(index, 'unit', e.target.value)}
          placeholder={t('productForm.priceUnitPlaceholder')}
        />
      ) : text || '-'
    ),
  },
  {
    title: t('common.actions'),
    width: 100,
    render: (_, record, index) => (
      editable ? (
        <Button
          type="link"
          danger
          onClick={() => handleDeletePrice(index)}
        >
          {t('common.delete')}
        </Button>
      ) : null
    ),
  },
];

// 使用
<BatchEditTable<PriceInfoDTO>
  columns={buildPricesColumns(detailEditable)}
  dataSource={pricesList}
  rowKey={(record, index) => `price-${index}`}
  onAddRow={() => {
    setPricesList(prev => [...prev, { price: 0, unit: '' }]);
  }}
  addRowLabel={t('productForm.addPrice')}
/>
```

---

### 3. 上传文档（documents）

**当前实现：** Upload + List 组件

**改造后：** BatchEditTable 组件

**表格列配置：**

| 列名 | 字段 | 类型 | 可编辑 | 宽度 |
|------|------|------|--------|------|
| 文档名称 | title | Input | ✅ | 300px |
| 文件大小 | size | Text | ❌ | 100px |
| 上传时间 | uploadTime | Text | ❌ | 150px |
| 操作 | - | Button | - | 150px |

**代码示例：**

```tsx
const buildDocumentsColumns = (editable: boolean): ColumnsType<ProductDocument> => [
  {
    title: t('productForm.documentName'),
    dataIndex: 'title',
    width: 300,
    render: (text, record, index) => (
      editable ? (
        <Input
          value={text}
          onChange={(e) => handleDocumentChange(index, 'title', e.target.value)}
          placeholder={t('productForm.documentNamePlaceholder')}
        />
      ) : text || '-'
    ),
  },
  {
    title: t('productForm.fileSize'),
    dataIndex: 'size',
    width: 100,
    render: (size) => size ? `${(size / 1024 / 1024).toFixed(2)} MB` : '-',
  },
  {
    title: t('productForm.uploadTime'),
    dataIndex: 'uploadTime',
    width: 150,
    render: (time) => time ? new Date(time).toLocaleString() : '-',
  },
  {
    title: t('common.actions'),
    width: 150,
    render: (_, record, index) => (
      <Space>
        <Button
          type="link"
          onClick={() => window.open(record.url, '_blank')}
        >
          {t('common.preview')}
        </Button>
        {editable && (
          <Button
            type="link"
            danger
            onClick={() => handleDeleteDocument(index)}
          >
            {t('common.delete')}
          </Button>
        )}
      </Space>
    ),
  },
];

// 使用
<div>
  {/* 上传按钮 */}
  {editable && (
    <Upload
      beforeUpload={handleDocumentUpload}
      showUploadList={false}
    >
      <Button icon={<UploadOutlined />}>
        {t('productForm.uploadDocument')}
      </Button>
    </Upload>
  )}
  
  {/* 文档列表 */}
  <BatchEditTable<ProductDocument>
    columns={buildDocumentsColumns(detailEditable)}
    dataSource={documentsList}
    rowKey={(record, index) => `doc-${index}`}
    style={{ marginTop: 16 }}
  />
</div>
```

---

## 📂 文件修改清单

### 需要修改的文件

1. **src/components/ProductForm.tsx**
   - 修改联系方式部分（phones）
   - 修改价格信息部分（prices）
   - 修改上传文档部分（documents）

2. **src/pages/ProductEditor.tsx**
   - 确保 ProductForm 的 props 传递正确
   - 确保状态管理逻辑正确

3. **src/locales/zh-CN.json** 和 **src/locales/en-US.json**
   - 添加新的翻译 key（如果需要）

### 可能需要新增的文件

无（复用现有的 `BatchEditTable` 组件）

---

## ✅ 验收标准

### 功能验收

1. **联系方式**
   - ✅ 可以添加新的联系方式
   - ✅ 可以编辑姓名和电话
   - ✅ 可以删除联系方式
   - ✅ 表格占据全宽度
   - ✅ 添加按钮在表格下方

2. **价格信息**
   - ✅ 可以添加新的价格信息
   - ✅ 可以编辑价格和单位
   - ✅ 可以删除价格信息
   - ✅ 价格输入框支持数字格式化
   - ✅ 表格占据全宽度
   - ✅ 添加按钮在表格下方

3. **上传文档**
   - ✅ 可以上传新的文档
   - ✅ 可以编辑文档名称
   - ✅ 可以预览文档
   - ✅ 可以删除文档
   - ✅ 显示文件大小和上传时间
   - ✅ 表格占据全宽度

### 视觉验收

1. ✅ 三个字段的样式与"产品用途管理"保持一致
2. ✅ 表格占据全宽度
3. ✅ 添加按钮在表格下方，使用虚线样式
4. ✅ 表格行高、字体大小、间距与产品用途管理一致
5. ✅ 响应式设计：移动端可以正常显示和操作

### 性能验收

1. ✅ 添加/删除操作响应时间 < 100ms
2. ✅ 表格渲染时间 < 200ms（100 条数据）
3. ✅ 文档上传进度显示正常

---

## 🧪 测试计划

### 单元测试

1. 测试联系方式的增删改功能
2. 测试价格信息的增删改功能
3. 测试文档上传、预览、删除功能

### 集成测试

1. 测试表单提交时，三个字段的数据格式正确
2. 测试只读模式下，三个字段的显示正确
3. 测试编辑模式下，三个字段的交互正确

### 用户测试

1. 邀请 2-3 名用户测试新的交互方式
2. 收集用户反馈，评估用户满意度
3. 根据反馈进行优化

---

## 📊 预期收益

### 用户体验提升

- **操作效率提升 30%**：批量操作、搜索功能
- **学习成本降低 40%**：统一的交互模式
- **视觉一致性提升 50%**：统一的布局风格

### 代码质量提升

- **代码复用性提升**：使用统一的 `BatchEditTable` 组件
- **维护成本降低**：统一的组件，统一的交互逻辑
- **可扩展性提升**：未来新增字段可以快速复用

---

## 🚀 实施计划

### 第一阶段：开发（1-2 天）

1. **Day 1**：
   - 上午：修改联系方式部分
   - 下午：修改价格信息部分

2. **Day 2**：
   - 上午：修改上传文档部分
   - 下午：自测 + 修复 bug

### 第二阶段：测试（0.5 天）

1. 单元测试
2. 集成测试
3. 用户测试

### 第三阶段：Review + 上线（0.5 天）

1. Code Review
2. 修复 Review 意见
3. 合并代码
4. 部署上线

---

## 📝 Review Checklist

### 代码质量

- [ ] 代码符合团队编码规范
- [ ] 没有硬编码的字符串（使用 i18n）
- [ ] 没有 console.log 等调试代码
- [ ] 变量命名清晰、语义化
- [ ] 函数职责单一，逻辑清晰

### 功能完整性

- [ ] 联系方式的增删改功能正常
- [ ] 价格信息的增删改功能正常
- [ ] 上传文档的上传、预览、删除功能正常
- [ ] 只读模式下显示正确
- [ ] 编辑模式下交互正确

### 视觉一致性

- [ ] 三个字段的样式与产品用途管理一致
- [ ] 表格占据全宽度
- [ ] 添加按钮在表格下方
- [ ] 响应式设计正常

### 性能

- [ ] 添加/删除操作响应时间 < 100ms
- [ ] 表格渲染时间 < 200ms（100 条数据）
- [ ] 文档上传进度显示正常

### 测试覆盖

- [ ] 单元测试覆盖率 > 80%
- [ ] 集成测试通过
- [ ] 用户测试反馈良好

---

## 📚 参考资料

1. **BatchEditTable 组件文档**：`src/components/common/BatchEditTable.tsx`
2. **产品用途管理实现**：`src/pages/ProductEditor.tsx` 第 1788-1842 行
3. **Ant Design Table 文档**：https://ant.design/components/table-cn/
4. **Ant Design Form 文档**：https://ant.design/components/form-cn/

---

## 🤝 协作说明

### 开发人员

1. 阅读本文档，理解改造方案
2. 按照技术实现部分的代码示例进行开发
3. 自测通过后，提交 Code Review
4. 根据 Review 意见修改代码
5. 合并代码，部署上线

### Review 人员

1. 检查代码质量（参考 Review Checklist）
2. 检查功能完整性
3. 检查视觉一致性
4. 检查性能
5. 提出修改意见
6. 确认修改后，批准合并

---

## 📞 联系方式

如有疑问，请联系：

- **产品经理**：[待填写]
- **技术负责人**：[待填写]
- **设计师**：[待填写]

---

**创建时间**：2026-05-03

**最后更新**：2026-05-03

**状态**：待开始

**优先级**：P1（高优先级）
