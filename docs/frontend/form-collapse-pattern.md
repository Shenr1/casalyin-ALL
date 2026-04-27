# 表单折叠面板设计模式

> 版本: v1.0  
> 创建时间: 2026-04-24  
> 适用范围: 所有内容详情表单（产品、店铺、品牌、农艺师等）

## 背景

长表单会给用户造成心理压力，尤其是在新增内容时。通过使用 Ant Design 的 Collapse 折叠面板，我们可以：
- 减少用户首次看到的表单长度
- 清晰地组织信息层级
- 保持单页流程，不需要来回切换
- 让用户快速跳到任意部分

## 设计原则

### 1. 分组原则

**按必填/选填分组：**
- 必填字段默认展开
- 选填字段默认折叠

**按相关性分组：**
- 将功能相关的字段放在同一个面板中
- 例如：价格信息和联系方式放在"价格与联系"面板

**分组数量：**
- 建议 5-7 个面板
- 避免过多分组（用户需要频繁展开）
- 避免过少分组（失去折叠的意义）

### 2. 默认展开/折叠规则

**默认展开的面板：**
- 包含必填字段的面板
- 用户最常填写的字段

**默认折叠的面板：**
- 只包含选填字段的面板
- 高级选项或管理员专属字段

### 3. 命名规范

面板标题应该：
- 简洁明了（2-6 个字）
- 描述该面板的内容类型
- 使用 i18n 翻译

## 实现示例

### 产品表单折叠面板结构

```tsx
<Collapse
  defaultActiveKey={['basic', 'tags']}
  style={{ marginBottom: 16 }}
>
  {/* 基础信息 - 默认展开 */}
  <Collapse.Panel header={t('productForm.basicInfo')} key="basic">
    {/* 产品名称（必填）、品牌（必填） */}
  </Collapse.Panel>

  {/* 标签与成分 - 默认展开 */}
  <Collapse.Panel header={t('productForm.tagsAndCompositions')} key="tags">
    {/* 标签信息（部分必填）、有效成分（必填） */}
  </Collapse.Panel>

  {/* 补充信息 - 默认折叠 */}
  <Collapse.Panel header={t('productForm.additionalInfo')} key="additional">
    {/* SKU、关联店铺、简述（选填） */}
  </Collapse.Panel>

  {/* 详细描述 - 默认折叠 */}
  <Collapse.Panel header={t('productForm.detailedDescription')} key="description">
    {/* 富文本编辑器（选填） */}
  </Collapse.Panel>

  {/* 价格与联系 - 默认折叠 */}
  <Collapse.Panel header={t('productForm.pricingAndContact')} key="pricing">
    {/* 价格信息、联系方式（选填） */}
  </Collapse.Panel>

  {/* 图片与文档 - 默认折叠 */}
  <Collapse.Panel header={t('productForm.mediaFiles')} key="media">
    {/* 产品图片、产品文档（选填） */}
  </Collapse.Panel>

  {/* 仓储与物流 - 默认折叠，仅 Admin */}
  {isAdmin && (
    <Collapse.Panel header={t('productForm.logistics')} key="logistics">
      {/* 可购城市、库存、Discourse Tag（选填） */}
    </Collapse.Panel>
  )}
</Collapse>
```

### 产品表单分组详情

| 面板 | Key | 默认状态 | 包含字段 | 必填字段 |
|------|-----|---------|---------|---------|
| 基础信息 | basic | 展开 | 产品名称、品牌 | 产品名称、品牌 |
| 标签与成分 | tags | 展开 | 标签信息、有效成分 | 必填标签、有效成分 |
| 补充信息 | additional | 折叠 | SKU、关联店铺、简述 | 无 |
| 详细描述 | description | 折叠 | 富文本编辑器 | 无 |
| 价格与联系 | pricing | 折叠 | 价格信息、联系方式 | 无 |
| 图片与文档 | media | 折叠 | 产品图片、产品文档 | 无 |
| 仓储与物流 | logistics | 折叠 | 可购城市、库存、Discourse Tag | 无 |

## 应用到其他表单

### 店铺表单（StoreForm）

建议分组：
1. **基础信息**（展开）- 店铺名称、品牌、联系方式
2. **位置信息**（展开）- 地址、地图坐标
3. **补充信息**（折叠）- 简述、详细描述
4. **图片与文档**（折叠）- 店铺图片、营业执照
5. **营业信息**（折叠）- 营业时间、服务范围

### 品牌表单（BrandForm）

建议分组：
1. **基础信息**（展开）- 品牌名称、公司名称
2. **联系信息**（展开）- 联系人、电话、邮箱
3. **品牌介绍**（折叠）- 简述、详细描述
4. **图片与文档**（折叠）- 品牌 Logo、营业执照
5. **认证信息**（折叠，仅 Admin）- 认证状态、审核信息

### 农艺师表单（AgronomistForm）

建议分组：
1. **基础信息**（展开）- 姓名、联系方式
2. **专业信息**（展开）- 专业领域、资质证书
3. **个人介绍**（折叠）- 简述、详细描述
4. **服务信息**（折叠）- 服务范围、服务价格
5. **图片与文档**（折叠）- 头像、证书照片

## i18n 翻译规范

每个折叠面板都需要添加对应的 i18n 翻译 key：

```json
// zh-CN.json
"xxxForm": {
  "basicInfo": "基础信息",
  "additionalInfo": "补充信息",
  "detailedDescription": "详细描述",
  // ... 其他面板标题
}

// en.json
"xxxForm": {
  "basicInfo": "Basic Information",
  "additionalInfo": "Additional Information",
  "detailedDescription": "Detailed Description",
  // ... 其他面板标题
}

// es-ES.json
"xxxForm": {
  "basicInfo": "Información Básica",
  "additionalInfo": "Información Adicional",
  "detailedDescription": "Descripción Detallada",
  // ... 其他面板标题
}
```

## 注意事项

### 1. 保持字段功能不变

折叠面板只是 UI 优化，不应该改变字段的功能：
- 必填字段的校验规则不变
- 字段的显示/隐藏逻辑不变
- 只读模式的行为不变

### 2. 隐藏字段的处理

隐藏字段（如 `brandDisplayName`、`customBrandName`）应该放在 Collapse 外面：

```tsx
</Collapse>

{/* 隐藏字段 */}
<Form.Item name="brandDisplayName" style={{ display: 'none' }}>
  <Input />
</Form.Item>
```

### 3. 条件渲染的面板

某些面板可能只在特定条件下显示（如管理员专属字段）：

```tsx
{isAdmin && (
  <Collapse.Panel header={t('xxxForm.adminSettings')} key="admin">
    {/* 管理员专属字段 */}
  </Collapse.Panel>
)}
```

### 4. 只读模式

在只读模式下，折叠面板仍然有效，用户可以展开/折叠查看内容。

### 5. 表单校验

折叠面板不影响表单校验：
- 必填字段的校验在提交时触发
- 即使字段在折叠的面板中，校验错误也会正常显示
- Ant Design 会自动展开包含错误字段的面板

## 优化效果

使用折叠面板后：
- ✅ 用户首次看到的表单长度减少 60-70%
- ✅ 必填字段更加突出
- ✅ 信息层级更加清晰
- ✅ 用户可以快速定位到需要填写的部分
- ✅ 保持单页流程，不需要来回切换

## 参考实现

完整的实现示例请参考：
- `casalyin-server/src/components/ProductForm.tsx` - 产品表单折叠面板实现
- `casalyin-server/src/pages/ProductEditor.tsx` - 如何传递 props 给表单组件

## 版本历史

- v1.0 (2026-04-24) - 初始版本，基于产品表单的优化经验
