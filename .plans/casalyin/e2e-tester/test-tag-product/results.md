# TC-TAG 产品 Tag 功能 E2E 测试结果

**测试日期：** 2026-04-21  
**测试账号：** COMPANY（123123@qq.com）  
**产品 ID：** 89（SKU: TG60415110，用于 TC-TAG-002/003 的测试数据）  
**测试文件：** `casalyin-server/e2e/v2.1/product-tag.spec.ts`

---

## 汇总

| 用例 | 状态 | 说明 |
|------|------|------|
| TC-TAG-001 | ✅ PASS（部分条件 SKIP） | Card 展开/Checkbox 可选已验证；isRequired/isMultiple 字段在 DB 中为 null，相关断言跳过 |
| TC-TAG-002 | ✅ PASS | 草稿编辑页 Corn tag 正确回填（`ant-checkbox-checked`） |
| TC-TAG-003 | ❌ FAIL — BUG | 后端 tag 数据正确，前端只读 tag 显示未触发 |
| TC-TAG-004 | ⏭ SKIP | DB 中无 isRequired=true 分类，无法测试 |

---

## 详细结果

### TC-TAG-001：产品创建流程

**环境：** 前端 5173 ✅，后端 8690 ✅（Tomcat 运行中）

| 检查项 | 结果 | 说明 |
|--------|------|------|
| tag 分类 Card 默认全部展开 | ✅ PASS | 27 个分类 Card 默认可见，无需手动展开 |
| isRequired 分类标题显示红色 * | ⏭ SKIP | DB 中 is_required 全部为 null，isRequired=true 的分类为 0 个 |
| isMultiple=false 使用 Radio.Group | ⏭ SKIP | DB 中 is_multiple 全部为 null，isMultiple=false 的分类为 0 个 |
| 选择 tag 后保存 | ✅ PASS | 点击 Corn Checkbox 后保存，API 返回 `{ok:true, data:97}`（草稿 ID） |

**草稿 ID 97 成功创建，tagIds 以 JSON 字符串 `"[3]"` 存储在草稿表。**

---

### TC-TAG-002：产品编辑 tag 回填

**测试路径：** `/products/edit-draft/97`（草稿编辑页）+ `/products/89`（发布后详情）

| 检查项 | 结果 | 说明 |
|--------|------|------|
| 草稿编辑页 Corn tag 已回填（Checkbox 勾选） | ✅ PASS | `ant-checkbox-checked` 存在，label 为「Corn」 |
| COMPANY 提交后免审直发 | ✅ PASS | `{ok:true, data:"提交成功，已直接发布"}` |
| 发布后产品详情页 tag 回填（Checkbox 模式） | ✅ PASS | `/products/89` 下 Corn Checkbox 已勾选 |

---

### TC-TAG-003：产品详情只读 tag 显示

**BUG 已确认：**

**表现：** `/products/89` 页面上，后端 API 返回 `tags: [{tagId:3, tagName:"Corn", categoryId:2, categoryName:"Crop Category"}]`，但页面上**没有**以「分类名：标签名」格式显示的 AntTag（只有「未知状态」状态标签）。

**根因：** `ProductEditor.tsx:180` 中：
```js
const [isEditMode] = useState(true)  // 永远为 true，没有 false 状态
```
导致 `ProductForm` 的 `readonly` prop：
```js
readonly={hasDualView || !isEditMode || editDisabledByLockOrPermission}
// = false || false || false = false
```
始终为 `false`，即使在 `mode='detail'`（`/products/:id`）下，ProductForm 也以**编辑模式**渲染，只读的 `productTags` AntTag 显示代码块（`readonly && ...`）从不执行。

**影响：** detail 模式下「分类名：标签名」格式的 tag 标签无法显示。

---

### TC-TAG-004：必填分类未选时提交报错

**跳过原因：** DB 中所有 tag 分类的 `is_required` 字段值为 `null`（非 `true`），无 `isRequired=true` 的分类，无法测试校验逻辑。后端返回 `isRequired: null`，前端 `.filter(cat => cat.isRequired)` 过滤后为空列表，故校验器不执行。

---

## Bug 清单

### BUG-TAG-001（MEDIUM）：isRequired / isMultiple 字段未在 ADMIN 后台配置

- **现象：** 27 个 tag 分类的 `is_required` 和 `is_multiple` 字段在 DB 中均为 `null`
- **后端接口：** `GET /tag/category/listAll` 返回的 `isRequired: null`、`isMultiple: null`
- **影响：** TC-TAG-001 无法验证红色 * 和 Radio 单选功能；TC-TAG-004 无法验证必填校验
- **建议：** ADMIN 在 tag 分类管理页配置至少一个 `isRequired=true` 的分类和一个 `isMultiple=false` 的分类，用于 E2E 验证

### BUG-TAG-002（HIGH）：ProductEditor detail 模式下 ProductForm readonly 永远为 false

- **位置：** `casalyin-server/src/pages/ProductEditor.tsx:180`
- **代码：** `const [isEditMode] = useState(true)` — 无切换到 false 的逻辑
- **现象：** `/products/:id`（detail 模式）页面显示可编辑的 Checkbox/Radio，而非只读 AntTag
- **影响：** 产品详情页 tag 以「分类名：标签名」格式显示的功能完全失效
- **预期：** detail 模式下 `readonly=true`，产品标签以 `<分类名>：<标签名>` AntTag 形式展示

---

## 正常行为记录

- tag 分类 Card 默认全部展开 ✅
- 草稿创建带 tag 正常（tagIds 写入 `[3]`）✅
- 发布后后端 `product/detail/:id` 正确 JOIN 返回 `tags` 数组 ✅
- 草稿编辑页 tag 回填正确（Checkbox 勾选状态恢复）✅
- COMPANY 提交草稿后免审直发 ✅
