# Editor 架构规范

> 合并自：editor_unified_spec.md + product_editor_spec.md
> 最后更新：2026-04-11

---

## 核心决策

四个业务维度（店铺、产品、品牌、农艺师）的新增/草稿编辑/详情页，**统一为单一 `XxxEditor` 组件**，通过 `mode` prop 区分行为，不拆成多个页面组件。

**Why：** 避免 create/edit/detail 三个页面重复维护相同逻辑（表单字段、权限判断、提交流程），保证 UI/交互在三种模式下完全一致。

**不存在独立的 `XxxCreate` / `XxxDetail` / `XxxDraftEdit` 页面组件**（旧文件已废弃或路由已 redirect）。

---

## 四个维度对应关系

| 维度 | 组件文件 | 路由 create | 路由 draft | 路由 detail |
|------|----------|-------------|------------|-------------|
| 店铺 | `StoreEditor.tsx` | `/stores/create` | `/stores/edit-draft/:id` | `/stores/:id` |
| 产品 | `ProductEditor.tsx` | `/products/create` | `/products/edit-draft/:id` | `/products/:id` |
| 品牌 | `BrandEditor.tsx` | `/brands/create` | `/brands/edit-draft/:id` | `/brands/:id` |
| 农艺师 | `AgronomistEditor.tsx` | `/agronomists/create` | `/agronomists/edit-draft/:id` | `/agronomists/:id` |

---

## 组件模式语义

```
mode = 'create'  → 新建（无 id）
mode = 'draft'   → 草稿编辑（有 draftId，从 useParams 取 id）
mode = 'detail'  → 已发布详情（有 entityId，从 useParams 取 id）
```

---

## 统一组件结构

```tsx
export default function XxxEditor({ mode }: { mode: 'create' | 'draft' | 'detail' }) {
  const { t } = useTranslation()
  const { modal } = App.useApp()          // 禁止使用静态 Modal.confirm
  const navigate = useNavigate()
  const { id: paramId } = useParams<{ id: string }>()
  const [form] = Form.useForm()

  // 角色判断：isAdmin 用权限码判断（见 ui-components.md）
  // state 声明
  // useEffect 加载数据
  // 各种 handler 函数（操作类 API 用动态 import）

  return (
    <ContentDetailLayout
      title={...}
      onBack={() => navigate('/xxx')}
      badge={...}
      isAdmin={isAdmin}
      onSave={...}
      onSaveAndPublish={...}
      publishLabel={...}
      meta={...}        // detail mode 展示元信息
      onOnline={isOnline ? undefined : handleOnline}
      onOffline={isOnline ? handleOffline : undefined}
    >
      {/* 表单内容 or Tabs */}
    </ContentDetailLayout>
  )
}
```

---

## 统一规范要点

1. **外层布局**：统一用 `ContentDetailLayout`，禁止自己写页面骨架
2. **Modal**：统一用 `App.useApp()` 返回的 `modal` 实例，禁止静态 `Modal.confirm`
3. **API 调用**：操作类 API 统一用动态 import（`await import('../api/xxx')`），避免首屏加载
4. **文案**：所有用户可见文案必须走 `t()`，禁止硬编码中文
5. **权限判断**：通过 permissionList 包含对应权限码来判断 `isAdmin`
6. **返回导航**：`onBack` 统一跳 `/{resource}` 列表页

---

## ProductEditor 额外规范

ProductEditor 相比其他维度有两处特殊：

### 产品用途表格（BatchEditTable）

- 产品详情页内嵌 `BatchEditTable` 组件，用于编辑产品的多维度用途（作物/虫害/剂量等）
- 表格数据独立保存，不走主表单的保存流程

### 关联店铺（showStores）

- detail mode 下展示关联店铺列表（Tabs 中的一个 Tab）
- 店铺展示逻辑：只显示 VIP 店铺（符合业务变现逻辑）

---

## StoreEditor 额外规范

### detail mode 关联产品 Tab（三期C，待开发）

- 关联产品列表：分页 Table + 取消关联 + 关联弹窗
- 状态：⏳ 待开发

---

## 改造完成度

| 维度 | 状态 |
|------|------|
| 品牌 BrandEditor | ✅ 完成 |
| 农艺师 AgronomistEditor | ✅ 完成 |
| 产品 ProductEditor | ✅ 完成 |
| 店铺 StoreEditor create/draft | ✅ 完成（三期A/B）|
| 店铺 StoreEditor detail + 关联产品Tab | ⏳ 待开发（三期C）|
