# 路由规范

> 来源：routes.md + editor_unified_spec.md（路由部分）
> 最后更新：2026-04-11（已根据 Editor 重构同步更新）

---

## 设计原则

四个业务维度统一遵循 REST 风格：

```
/:resource                     列表页
/:resource/create              新建页（XxxEditor mode="create"）
/:resource/:id                 详情页（XxxEditor mode="detail"）
/:resource/edit-draft/:id      草稿编辑页（XxxEditor mode="draft"）
```

**附加规则：**
- 路径全小写 kebab-case
- 动态参数统一用 `:id`，不带业务前缀（不用 `:storeId`、`:productId`、`:draftId`）
- 子模块必须归属父命名空间（如品牌申请在 `/brands/apply/...`，VIP 相关在 `/vip/...`）
- 菜单配置（`config/routes.tsx`）和路由注册（`routes/index.tsx`）必须严格对应
- 旧路由保留为 redirect，不裸删

---

## 完整路由清单

### 认证页（无 Layout）

| 路由 | 页面 | 说明 |
|------|------|------|
| `/login` | Login | 登录 |
| `/register` | Register | 注册 |
| `/forgot-password` | ForgotPassword | 找回密码 |
| `/privacy` | PolicyPlaceholder | 隐私政策 |
| `/terms` | PolicyPlaceholder | 服务条款 |
| `/internal/auth-admin-portal` | AdminLogin | 隐藏管理员登录入口 |

### 控制台

| 路由 | 页面 |
|------|------|
| `/dashboard` | Dashboard |

### 店铺模块 `/stores`

| 路由 | 组件 | 说明 |
|------|------|------|
| `/stores` | StoreManagement | 列表 |
| `/stores/create` | `<StoreEditor mode="create" />` | 新建 |
| `/stores/:id` | `<StoreEditor mode="detail" />` | 详情 |
| `/stores/edit-draft/:id` | `<StoreEditor mode="draft" />` | 草稿编辑 |

旧路由（redirect）：
- `/stores/detail/:storeId` → `/stores/:id`
- `/stores/edit/:storeId` → `/stores/:id`

### 产品模块 `/products`

| 路由 | 组件 | 说明 |
|------|------|------|
| `/products` | ProductList | 列表 |
| `/products/create` | `<ProductEditor mode="create" />` | 新建 |
| `/products/:id` | `<ProductEditor mode="detail" />` | 详情 |
| `/products/edit-draft/:id` | `<ProductEditor mode="draft" />` | 草稿编辑 |
| `/products/audit-status` | ProductAuditStatus | 审核状态查询 |

旧路由（redirect）：
- `/products/detail/:productId` → `/products/:id`

### 农艺师模块 `/agronomists`

| 路由 | 组件 | 说明 |
|------|------|------|
| `/agronomists` | AgronomistManagement | 列表 |
| `/agronomists/create` | `<AgronomistEditor mode="create" />` | 新建 |
| `/agronomists/:id` | `<AgronomistEditor mode="detail" />` | 详情 |
| `/agronomists/edit-draft/:id` | `<AgronomistEditor mode="draft" />` | 草稿编辑 |

旧路由（redirect）：
- `/agronomists/detail/:agronomistId` → `/agronomists/:id`
- `/agronomist/profile` → `/agronomists/create`

### 品牌模块 `/brands`

| 路由 | 组件 | 说明 |
|------|------|------|
| `/brands` | BrandPortal | 品牌入口页 |
| `/brands/apply-my` | BrandApplyMyPage | 用户品牌认证申请中心（BASE/COMPANY）|
| `/brands/list` | BrandManagement | 列表（需权限）|
| `/brands/create` | `<BrandEditor mode="create" />` | 新建 |
| `/brands/:id` | `<BrandEditor mode="detail" />` | 详情/编辑 |
| `/brands/edit-draft/:id` | `<BrandEditor mode="draft" />` | 草稿编辑 |
| `/brands/apply` | BrandApplyManagement | 申请列表 |
| `/brands/apply/:id` | BrandApplyDetail | 申请详情 |

旧路由（redirect）：
- `/brands/management` → `/brands/list`
- `/brands/:id/edit` → `/brands/:id`
- `/brands/draft/:id` → `/brands/edit-draft/:id`
- `/brand-apply` → `/brands/apply`
- `/brand-apply/:applyId` → `/brands/apply/:id`

### 基础数据模块 `/tags`

| 路由 | 页面 |
|------|------|
| `/tags` | → redirect `/tags/category` |
| `/tags/category` | TagCategoryManagement |
| `/tags/pest` | PestManagement |
| `/tags/crop` | CropManagement |
| `/tags/composition` | CompositionManagement |

> 菜单名叫「基础数据」，路由用 `/tags` 是历史原因，暂不改。

旧路由（redirect）：`/pests`、`/crops`、`/compositions`

### VIP 模块 `/vip`

| 路由 | 页面 | 权限 |
|------|------|------|
| `/vip` | → redirect `/vip/subscriptions` | — |
| `/vip/my` | MyVipStatus（我的 VIP）| 登录即可 |
| `/vip/subscriptions` | Subscriptions（订阅管理）| ADMIN |
| `/vip/subscriptions/batch-add` | BatchSetVip（批量设置 VIP）| ADMIN |
| `/vip/levels` | VipLevelManagement（等级管理）| ADMIN |
| `/vip/contact` | ContactUs | ADMIN |

旧路由（redirect）：
- `/subscriptions` → `/vip/subscriptions`
- `/subscriptions/batch-add` → `/vip/subscriptions/batch-add`
- `/vip-levels` → `/vip/levels`

### 系统管理 `/system`（ADMIN only）

| 路由 | 页面 |
|------|------|
| `/system` | → redirect `/system/users` |
| `/system/users` | SystemUsers |
| `/system/role-guide` | SystemRoles |
| `/system/permissions` | SystemPermissions |
| `/system/audit-logs` | SystemAuditLogs |

### 内容日志

| 路由 | 页面 | 权限 |
|------|------|------|
| `/content-logs` | ContentLogs | ADMIN |
| `/content-logs/:id` | ContentLogDetail | ADMIN |

### 其他

| 路由 | 页面 |
|------|------|
| `/search` | GlobalSearch（ADMIN only）|
| `/profile` | CustomerProfile |
| `/notifications` | Notifications |

---

## 新增页面操作流程

1. 在 `src/pages/` 下创建页面组件（遵循 XxxEditor 三合一架构）
2. 在 `src/routes/index.tsx` 按命名空间注册路由，参数统一用 `:id`
3. 如需出现在菜单，在 `src/config/routes.tsx` 对应位置添加菜单项
4. 更新本文件的路由清单
