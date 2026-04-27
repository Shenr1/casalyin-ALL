# ProductForm.tsx 重构指南 - 2个折叠面板

> 本文档提供详细的重构步骤，将 ProductForm.tsx 从4个折叠面板重构为2个折叠面板。

## 目标

将产品表单重新组织为 **2个折叠面板**：
1. **基础配置**（必填项）- 默认展开
2. **高级配置**（选填项）- 默认折叠

## 当前结构（4个面板）

| 面板 | key | 默认状态 | 内容 |
|------|-----|---------|------|
| 基础配置 | basic | 展开 | 产品名称、品牌、价格列表、联系人列表、店铺、简介 |
| 标签与成分 | tags | 折叠 | 标签分类（必选+非必选）、有效成分 |
| 详细描述 | description | 折叠 | SKU、产品描述、产品图片、产品文档 |
| 仓储与物流 | logistics | 折叠 | 可购城市、库存数量、Discourse 标签（仅 Admin）|

## 目标结构（2个面板）

| 面板 | key | 默认状态 | 内容 |
|------|-----|---------|------|
| 基础配置 | basic | 展开 | 产品名称、品牌、有效成分、必选标签分类 |
| 高级配置 | advanced | 折叠 | SKU、店铺、简介、价格、联系人、非必选标签、描述、图片、文档、仓储物流 |

## 重构步骤

### 步骤 1：修改 Collapse 的 defaultActiveKey

**位置：** Line 233

**修改前：**
```typescript
<Collapse
  defaultActiveKey={['basic']}
  style={{ marginBottom: 16 }}
  bordered={false}
>
```

**修改后：** 保持不变（已经是 `['basic']`）

---

### 步骤 2：重构"基础配置"面板 - 移除选填项

**位置：** Line 238-467

**需要移除的内容：**
1. Line 339-355：店铺选择（showStores）
2. Line 357-368：产品简介（info）
3. Line 370-417：价格列表（prices）
4. Line 419-466：联系人列表（contacts）

**保留的内容：**
1. Line 241-250：产品名称（name）
2. Line 252-338：品牌（brandId）

**需要添加的内容（从"标签与成分"面板移动过来）：**
1. 有效成分（compositionIds）- 从 Line 760-795
2. 必选标签分类 - 从 Line 543-649（只保留 `isRequired === true` 的部分）

---

### 步骤 3：创建"高级配置"面板

**位置：** Line 469（替换"标签与成分"面板的 header）

**修改前：**
```typescript
{/* 标签与成分 - 默认展开 */}
<Collapse.Panel header={t('productForm.tagsAndCompositions')} key="tags">
```

**修改后：**
```typescript
{/* ========== 高级配置（选填项）- 默认折叠 ========== */}
<Collapse.Panel header={t('productForm.advancedConfig')} key="advanced">
```

---

### 步骤 4：重组"高级配置"面板内容

**"高级配置"面板应包含（按顺序）：**

1. **SKU** - 从 Line 800-805 移动过来
2. **店铺选择** - 从 Line 339-355 移动过来
3. **产品简介** - 从 Line 357-368 移动过来
4. **价格列表** - 从 Line 370-417 移动过来
5. **联系人列表** - 从 Line 419-466 移动过来
6. **非必选标签分类** - 保留 Line 651-711（只保留 `isRequired === false` 的部分）
7. **产品描述** - 从 Line 807-813 移动过来
8. **产品图片** - 从 Line 815-834 移动过来
9. **产品文档** - 从 Line 836-902 移动过来
10. **仓储与物流**（Admin）- 从 Line 905-945 移动过来

---

### 步骤 5：删除"详细描述"面板

**位置：** Line 798-903

**操作：** 删除整个 `<Collapse.Panel header={t('productForm.detailedDescription')} key="description">` 面板

---

### 步骤 6：删除"仓储与物流"面板

**位置：** Line 905-946

**操作：** 删除整个 `<Collapse.Panel header={t('productForm.storageAndLogistics')} key="logistics">` 面板（内容已移到"高级配置"）

---

### 步骤 7：更新 i18n 文案

**删除不再使用的 key：**

在 `src/locales/zh-CN.json`、`src/locales/en.json`、`src/locales/es-ES.json` 中删除：
- `productForm.tagsAndCompositions`
- `productForm.detailedDescription`
- `productForm.storageAndLogistics`

**确认已存在的 key：**
- `productForm.basicConfig` ✅
- `productForm.advancedConfig` ✅

---

## 详细代码块移动清单

### 从"基础配置"移到"高级配置"

| 字段 | 当前位置 | 目标位置 | 代码行数 |
|------|---------|---------|---------|
| 店铺选择（stores） | Line 339-355 | "高级配置"面板开头 | 17行 |
| 产品简介（info） | Line 357-368 | 店铺选择之后 | 12行 |
| 价格列表（prices） | Line 370-417 | 产品简介之后 | 48行 |
| 联系人列表（contacts） | Line 419-466 | 价格列表之后 | 48行 |

### 从"标签与成分"移到"基础配置"

| 字段 | 当前位置 | 目标位置 | 代码行数 |
|------|---------|---------|---------|
| 有效成分（compositionIds） | Line 760-795 | 品牌之后 | 36行 |
| 必选标签分类 | Line 543-649 | 有效成分之后 | ~107行（需要过滤 `isRequired === true`）|

### 从"详细描述"移到"高级配置"

| 字段 | 当前位置 | 目标位置 | 代码行数 |
|------|---------|---------|---------|
| SKU | Line 800-805 | "高级配置"面板开头 | 6行 |
| 产品描述（description） | Line 807-813 | 非必选标签之后 | 7行 |
| 产品图片（pictures） | Line 815-834 | 产品描述之后 | 20行 |
| 产品文档（documents） | Line 836-902 | 产品图片之后 | 67行 |

### 从"仓储与物流"移到"高级配置"

| 字段 | 当前位置 | 目标位置 | 代码行数 |
|------|---------|---------|---------|
| 可购城市（availableCities） | Line 909-921 | 产品文档之后 | 13行 |
| 库存数量（stockQuantity） | Line 923-936 | 可购城市之后 | 14行 |
| Discourse 标签（discourseTag） | Line 938-944 | 库存数量之后 | 7行 |

---

## 标签分类逻辑调整

### 必选标签分类（放在"基础配置"）

**位置：** Line 592-649

**过滤条件：**
```typescript
tagCategories
  .filter(category => !category.disabledFlag && category.isRequired)
  .map(category => { /* 渲染逻辑 */ })
```

### 非必选标签分类（放在"高级配置"）

**位置：** Line 651-711

**保持现有逻辑：**
- 已添加的非必选标签分类（addedCategories）
- 添加标签分类按钮
- 添加标签分类的 Modal

---

## 验证清单

完成重构后，请验证以下内容：

### 功能验证

- [ ] "基础配置"面板默认展开
- [ ] "高级配置"面板默认折叠
- [ ] 产品名称、品牌、有效成分、必选标签在"基础配置"中
- [ ] SKU、店铺、简介、价格、联系人、非必选标签、描述、图片、文档在"高级配置"中
- [ ] Admin 用户可以看到仓储与物流字段（在"高级配置"中）
- [ ] 必填项验证正常工作
- [ ] 标签分类按 isRequired 正确分离

### TypeScript 编译

```bash
cd casalyin-server
npm run type-check
```

应该 0 错误。

### 测试

1. **创建产品测试**：
   - 只填写必填字段（产品名称、品牌、有效成分、必选标签）
   - 确认可以成功创建产品
   - 确认"高级配置"默认折叠

2. **编辑产品测试**：
   - 打开已有产品
   - 确认必填字段在"基础配置"中
   - 确认选填字段在"高级配置"中

3. **只读模式测试**：
   - 查看产品详情
   - 确认所有字段正确显示

---

## 预计时间

- 执行重构：20-30 分钟
- 验证测试：10-15 分钟
- 总计：30-45 分钟

---

## 注意事项

1. **保持现有逻辑**：
   - 不要修改字段的验证规则
   - 不要修改字段的渲染逻辑
   - 只是移动字段的位置

2. **品牌字段的特殊处理**：
   - 需要保留 companyBrand 的判断逻辑
   - 需要保留 BrandSelect 组件的所有逻辑
   - 需要保留隐藏字段（customBrandName、brandDisplayName）

3. **只读模式的处理**：
   - 保持 readonly 参数的所有逻辑
   - 不要修改只读模式下的显示方式

4. **标签分类的分离**：
   - 必选标签分类（isRequired === true）放在"基础配置"
   - 非必选标签分类（isRequired === false）放在"高级配置"
   - 保持添加/删除标签分类的逻辑不变

---

## 完成后

1. 更新 `progress.md`，记录重构完成
2. 通知 reviewer 进行代码审查
3. 运行 E2E 测试验证功能正常
