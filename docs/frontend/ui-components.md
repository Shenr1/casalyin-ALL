# 前端 UI 与组件规范

> 合并自：frontend_ui.md + feedback_antd_modal.md
> 最后更新：2026-04-11

---

## 页面宽度

**列表页**：使用 ProLayout 默认宽度（`maxWidth: 1440px`），不需要额外包裹容器。

**详情页（ContentDetailLayout）**：组件内部已限制内容区宽度，不需要额外设置。

不要用固定的 `maxWidth: 720` 或 `maxWidth: 860` 等小宽度。

> 历史规范「所有页面 maxWidth: 1000px」已废弃（2026-04-12），ProLayout 默认值即为规范。

---

## 返回按钮

- **只放图标，不加文字**：`<Button icon={<ArrowLeftOutlined />} onClick={...} />`
- 放在 Card `title` 最左侧（用 Space 包裹图标 + 标题文字）
- 不放在 `extra`（extra 是右上角）
- **不出现「返回」二字**
- **统一无 type**（默认 outlined，有边框），不用 `type="text"` 或 `type="link"`
- 页面顶部有返回箭头时，底部操作栏**不再出现「取消」按钮**
- 详情页内**不出现「返回列表」按钮**

---

## 详情页布局：ContentDetailLayout

> 所有维度详情/编辑/新建页统一使用此组件，禁止自己写页面骨架，SaveBar 已废弃。
> 组件位置：`src/components/ContentDetailLayout.tsx`

### 整体结构

```
┌──────────────────────────────────────────────────────────────────────┐
│ Header: [←]  [页面标题]  [状态Badge]  ───  [👁]  [保存]  [保存并发布] │
└──────────────────────────────────────────────────────────────────────┘
│  主体表单（~70%）                           │  Sidebar（~30%）        │
│  [Form fields...]                          │  📋 内容信息            │
│                                            │  创建时间/发布时间/所属人 │
│                                            │  ────────────────       │
│                                            │  ⚙️ 操作                │
│                                            │  [认证/取消认证]        │
│                                            │  [上架/下架]            │
│                                            │  [发整改通知]           │
│                                            │  ────────────────       │
│                                            │  [删除] (danger, 仅A)   │
└────────────────────────────────────────────┴────────────────────────┘
```

### Header 规则

**左侧（固定）：** `[← 返回图标]` `[页面标题]` `[状态 Badge/Tag]`

**右侧主操作（按角色区分）：**
- `[👁 预览图标]`（始终渲染，无 previewUrl 时 disabled）
- BC 角色：`[保存]` `[保存并发布]`
- A 角色（isAdmin）：`[保存并发布]`（无单独保存，Admin 直接发布）

### Sidebar 操作权限

| 操作 | A 角色 | BC 角色 |
|------|--------|---------|
| 认证/取消认证 | ✅ | ❌ |
| 上架/下架 | ✅ | ✅（自己内容）|
| 发整改通知 | ✅ | ❌ |
| 删除（物理删除）| ✅ | ❌ |

### isAdmin 判断规范

统一用权限码判断，不用 roleTypes：
- 产品：`permissionList.includes('product:audit')`
- 品牌：`permissionList.includes('brand:audit')`
- 店铺：`permissionList.includes('store:audit')`
- 农艺师：`permissionList.includes('agronomist:audit')`

---

## 状态 Tag 颜色规范

```
status=0（待审核/草稿）→ color="orange"
status=1（已发布）     → color="green"
status=2（已拒绝）     → color="red"
其他/未知              → color="default"
```

不使用 `processing`、`blue`、`error` 等其他颜色。

---

## 表单字段布局

- **单个 Input 不独占一整行**，应分列放置
- 短字段（姓名、手机、职称、单位等）放 `<Row gutter={24}>` 两列，每列 `span={12}`
- 长字段（URL、地址、描述单行等）可单独一行，但限制在 `maxWidth: '50%'` 或 `Col span={12}`
- TextArea、RichTextEditor 可独占一行
- Select multiple 可独占一行或两列并排
- 同一语义分组的字段放在同一 Row 里

---

## Table 列规范

- **所有 table 不出现【操作】列**
- 查看详情、编辑均通过点击行（`onRow onClick`）进入详情页

### 已发布列表各维度列定义

**产品：** 产品名称 / 品牌 / 成分（第1个+hover全部）/ VIP等级 / 状态 / 认证状态 / 归属用户（仅Admin）

**品牌：** 品牌名称 / VIP等级 / 状态 / 认证状态 / 归属用户（仅Admin）（品牌代码移到详情页）

**店铺：** 店铺名称 / 地址（省·市）/ VIP等级 / 状态 / 认证状态 / 归属用户（仅Admin）

**农艺师：** 姓名 / 职称 / 所属机构 / VIP等级 / 状态 / 认证状态 / 归属用户（仅Admin）

### 未发布列表（四个维度统一）

名称 / 操作类型（新增/修改 Tag）/ 审核状态（草稿/待审核/已拒绝 Tag）/ 拒绝原因（仅已拒绝）/ 提交时间 / 提交人（仅Admin）

### 移到详情页的内容（不在列表展示）

认证操作按钮、下架/上架操作按钮、创建时间、更新时间、品牌代码、联系方式、创建者类型

---

## 详情页编辑

- 进入详情页后可直接编辑，不需要单独的【编辑】按钮
- **保存操作统一在底部 StickyFooterBar**（禁止在 Header 右上角放保存按钮）

---

## 底部操作栏：StickyFooterBar

> 组件位置：`src/components/StickyFooterBar.tsx`  
> 必须通过 `ContentDetailLayout` 的 `footerBar` prop 传入，不可单独放置。

### 功能

- **左侧**：显示「上次保存时间」（尚未保存 / 刚刚 / X 分钟前，每分钟自动刷新）
- **右侧**：`[取消（可选）]` `[extraActions（可选，用于发布/提交审核）]` `[保存]`

### Props

```tsx
interface StickyFooterBarProps {
  lastSavedAt?: Date | null   // null = 从未保存
  saving?: boolean            // 主按钮 loading 态
  onSave?: () => void         // 保存回调（不传则不渲染保存按钮）
  onCancel?: () => void       // 取消回调（不传则不渲染取消按钮）
  extraActions?: ReactNode    // 「提交审核」「发布」等额外按钮（插在保存左侧）
  saveText?: string           // 主按钮文案，默认 t('common.save')
}
```

### 使用方式

```tsx
// 1. 在 Editor 组件内新增 state
const [lastSavedAt, setLastSavedAt] = useState<Date | null>(null)

// 2. 保存成功后更新时间
setLastSavedAt(new Date())

// 3. 构建 footerBar
const footerBar = (
  <StickyFooterBar
    lastSavedAt={lastSavedAt}
    saving={saving}
    onSave={handleSave}
    extraActions={
      <Button type="primary" onClick={handlePublish}>发布</Button>
    }
  />
)

// 4. 传入 ContentDetailLayout
<ContentDetailLayout footerBar={footerBar} ...>
  {children}
</ContentDetailLayout>
```

### 规范约束

- 所有详情/编辑/新建页**必须**用 StickyFooterBar 承载保存操作
- **禁止**在 Header 右上角放「保存」或「保存并发布」按钮
- `lastSavedAt` 更新时机：保存 API 返回成功后，调用 `setLastSavedAt(new Date())`
- i18n key：`common.neverSaved` / `common.lastSavedAt` / `common.justNow`（三语言已同步）

---

## 前端权限控制

- 在 `App.tsx` 的 `filterRoutesByPermission` 函数中实现
- 读取登录返回的 `permissionCodeList` 和 `roleTypes`
- `access/index.ts` 已删除（废弃文件）

路由权限字段（`routes.tsx`）：
- `accessPermission`：需要有对应权限码才显示菜单
- `showOnlyForRoles`：只对指定角色显示
- `hideForRoles`：对指定角色隐藏

---

## Modal / message / notification 使用规范

> **禁止使用 Ant Design 静态方法**，必须使用 `AppProvider` 导出的实例。

**Why：** Ant Design 5.x 的 `Modal.confirm` 等静态方法在有 `ConfigProvider`/`App` 包裹的项目中会丢失 context，调用后无任何响应、无报错，极难排查。

```ts
// ❌ 错误
import { Modal, message } from 'antd'
Modal.confirm({ ... })
message.success('...')

// ✅ 正确
import { modal, message } from '../components/AppProvider'
modal.confirm({ ... })
message.success('...')
```

`modal`、`message`、`notification` 均统一从 `AppProvider`（`src/components/AppProvider.tsx`）导入。
