# Task: Tag UX Refactor — findings

## 改动摘要

将 ProductForm 的标签选择区域从「两步式（先选分类→再选标签）」重构为「单层展开式（所有分类默认展开）」。

## 文件改动

### `casalyin-server/src/api/tag.ts`
- `TagCategory` 接口新增 `isRequired?: boolean` 和 `isMultiple?: boolean` 两个可选字段

### `casalyin-server/src/components/ProductForm.tsx`
- import 行：新增 `Radio`，移除 `CloseOutlined`（不再使用）
- 删除状态：`selectedCategoryIds`（不再需要分类选择步骤）
- 删除函数：`getTagIdsArray`、`handleCategoryChange`（两个已无用）
- 修改 useEffect：原来监听 `selectedCategoryIds` 按需加载，改为在 `tagCategories` 变化时立即加载所有非禁用分类的标签
- 重写标签区域 UI：
  - 只读模式：只展示已选标签 tag（逻辑不变，代码简化）
  - 编辑模式：去掉第一步分类 Checkbox.Group，所有分类直接展开为 Card
    - isMultiple === false → Radio.Group（单选）
    - 其余 → Checkbox.Group（多选）
    - isRequired 为 true 时 Card 标题显示红色 *

## TypeScript 状态
tsc --noEmit 零错误通过。
