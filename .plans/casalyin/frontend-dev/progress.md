# frontend-dev - 工作日志

> 用于上下文恢复。压缩/重启后先读此文件。

---

## 2026-04-11 Session 1 — 初始化

- [x] 工作目录初始化完成
- [x] 等待 team-lead 分配任务

### 菜单项添加（T3 补充）
- [x] `config/routes.tsx`: 新增 `/brands/apply-my` 菜单项，`SafetyCertificateOutlined` 图标，`accessPermission: brand:apply`，`hideForRoles: [ADMIN]`
- [x] i18n 三语言 `menu.brandApplyMy`: 品牌认证申请 / Brand Certification / Certificación de Marca
- [x] TypeScript 编译通过

## 2026-04-13 Session — 品牌管理页面三处改动

### 改动1：菜单重构 routes.tsx ✅
- 删除 /brands/apply-my 独立菜单项
- 新增 BASE/COMPANY 单入口（hideForRoles: ['ADMIN']）
- 新增 ADMIN 二级菜单（showOnlyForRoles: ['ADMIN']），子项 /brands/list 和 /brands/apply

### 改动2：BrandDashboard 产品列表区块 ✅
- brand.ts 新增 BrandProductItem 接口 + apiGetBrandProducts 占位函数
- BrandDashboard.tsx 新增产品列表 Table（4列：封面/名称/VIP/状态，点击行跳转/products/:id）
- 新增流量数据占位 Card（灰色文字"即将上线"）

### 改动3：i18n 三语言同步 ✅
- 新增 menu.brand / menu.brandList / menu.brandApply 三个 key（zh-CN / en / es-ES）
- 新增 brandDashboard.productList.* 和 brandDashboard.trafficData.* 共 12 个 key

TypeScript 编译无错误。

## 2026-04-14 Session — v1.3 reviewer 问题修复确认

**任务**：task #1（v1.3 前端 BrandDashboard + reviewer 问题修复）

确认上一次 commit（228a3dd, 2026-04-13）已包含所有 reviewer 指出的修复：
- MEDIUM-1：接口路径 `/brand/my-products` 与后端 Controller 一致 ✅
- MEDIUM-2：productsError 状态 + Table locale.emptyText 显示错误提示 ✅
- MEDIUM-3：ADMIN 父节点虚拟路径 `/brands/_group` ✅
- LOW-1：移除 `|| '刷新'` 硬编码兜底，common.refresh 三语言均存在 ✅

TypeScript 编译通过（0 错误）。
已更新 findings.md，向 reviewer 发起二次审查请求。

## 2026-04-14 阻塞记录

e2e-tester 发现 HIGH Bug TC-P3-001：ADMIN「认证申请」子菜单不显示。
根因：后端 ADMIN 角色缺少 `brand:apply:audit` 权限码（非前端问题）。
决策：方案 A，backend-dev 补充权限码，前端 routes.tsx:53 不动。
**前端无需操作，等待 backend-dev 修复后 e2e 回归结果。**

## 2026-04-25 Session — SKU 字段状态调查

**任务**：调查 SKU 字段是否被移除，导致测试失败

**调查结果**：
- SKU 字段仍然存在，但已从必填改为选填
- 位置：从"基础信息"第一行移至"详细描述"折叠面板中（默认折叠）
- 变更时间：产品表单 UI 优化（第三期、第四期）期间
- ProductEditor 中的 SKU 代码不是遗留代码，字段仍正常工作

**决策**：采用方案 A（跳过 SKU）
- 测试脚本不填写 SKU 字段（已非必填）
- 减少测试复杂度，避免展开折叠面板

**状态**：已完成调查并汇报 team-lead

## 2026-04-25 Session — 致命 Bug 修复：必填标签分类无可选标签

**任务**：修复产品创建失败的致命 bug

**问题**：
- 标签分类被设置为必填（isRequired=true）
- 但该分类下没有任何可选标签
- 导致用户无法提交产品创建表单

**修复内容**：

1. **增强验证逻辑**（ProductForm.tsx:508-528）
   - 在表单验证器中优先检测"必填分类但无可选标签"的情况
   - 给出明确的错误提示

2. **视觉反馈**（ProductForm.tsx:557-598）
   - 检测致命问题：`isBlocker = category.isRequired && hasNoTags`
   - Select 组件添加 `status="error"` 红色边框
   - Form.Item 添加 `validateStatus="error"` 和 `help` 错误提示
   - 禁用有问题的 Select
   - placeholder 显示"该分类下暂无标签"

3. **i18n 文案**（三语言同步）
   - 新增 `requiredCategoryNoTags` key
   - zh-CN: "必选分类\"{{name}}\"下没有可选标签，请联系管理员添加标签后再创建产品"
   - en: "Required category \"{{name}}\" has no available tags. Please contact the administrator to add tags before creating products"
   - es-ES: "La categoría obligatoria \"{{name}}\" no tiene etiquetas disponibles. Póngase en contacto con el administrador para agregar etiquetas antes de crear productos"

**修改文件**：
- `src/components/ProductForm.tsx`
- `src/locales/zh-CN.json`
- `src/locales/en.json`
- `src/locales/es-ES.json`

**TypeScript 编译**：✅ 通过（0 错误）

**状态**：已完成修复并汇报 team-lead，等待 e2e-tester 测试验证

## 2026-04-25 Session — 待办：产品列表接口分离

**任务**：等待 backend-dev 完成后端接口后，修改产品列表页面

**背景**：
- 用户确认采用方案 B（分离查询接口）修复产品列表显示问题

**待修改内容**：

1. **未发布 Tab**（ProductList.tsx:163-195）
   - 当前：使用 `apiGetAllProductDrafts()` / `apiGetMyProductDrafts()`
   - 修改为：使用新接口 `/product/draft/query`
   - 接口参数：
     - `draftStatus`（0-草稿, 1-待审核, 2-已拒绝）
     - 支持分页（pageNum, pageSize）
     - 支持搜索和排序

2. **已发布 Tab**（ProductList.tsx:98-161）
   - 保持使用 `/product/query` 接口
   - 确认显示所有已发布的产品（不管上架还是下架）

**准备工作**：
- 已检查 ProductList.tsx 代码结构
- 已了解当前的 Tab 结构和接口调用逻辑
- refreshDrafts() 函数位于 line 163-195
- refreshPublished() 函数位于 line 98-161

**状态**：等待 backend-dev 完成后端接口，team-lead 通知后开始修改

## 2026-04-25 Session — 产品列表接口分离（已完成）

**任务**：修改产品列表页面，使用新的草稿查询接口

**背景**：
- 用户确认采用方案 B（分离查询接口）修复产品列表显示问题
- backend-dev 已完成后端接口 `/product/draft/query`

**修改内容**：

### 1. 修改 refreshDrafts 函数（ProductList.tsx:163-207）

**变更前**：
- 使用 `apiGetAllProductDrafts()` / `apiGetMyProductDrafts()`
- 不支持分页、搜索、排序
- 返回所有草稿数据

**变更后**：
- 使用 `apiQueryProductDrafts()` 接口（对应后端 `/product/draft/query`）
- 支持分页参数：`pageNum`, `pageSize`
- 支持搜索参数：`keyword`
- 支持排序参数：`sortColumn`
- Admin 用户：不传 `creatorId`（查询所有用户的草稿）
- 普通用户：传 `creatorId`（只查询自己的草稿）

**接口参数**：
```typescript
{
  pageNum,
  pageSize,
  searchCount: true,
  sortItemList: [{ isAsc: false, column: sortColumn || sortByRef.current }],
  keyword: keyword?.trim() || undefined,
  creatorId: isAdmin ? undefined : userId || undefined,
}
```

**响应处理**：
- 从 `result.list` 获取草稿列表
- 从 `result.total` 获取总数，更新分页状态
- 从 `result.draftCount`, `result.pendingCount`, `result.rejectedCount` 计算未发布总数
- 更新 Tab 计数显示

### 2. 已发布 Tab 保持不变

`refreshPublished` 函数（line 98-161）保持使用 `/product/query` 接口，显示所有已发布的产品（包括上架和下架）。

### 3. 函数签名统一

`refreshDrafts` 和 `refreshPublished` 现在具有相同的函数签名：
```typescript
(pageNum?: number, pageSize?: number, keyword?: string, sortColumn?: string) => Promise<void>
```

这样 `refresh` 函数（line 197-209）可以统一调用两者。

**修改文件**：
- `src/pages/ProductList.tsx`

**TypeScript 编译**：✅ 通过（0 错误）

**状态**：已完成修改并汇报 team-lead，等待测试验证
