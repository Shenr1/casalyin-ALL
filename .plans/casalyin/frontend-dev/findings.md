# frontend-dev - 发现索引

> 纯索引——每个条目应简短（Status + Report 链接 + Summary）。

---

（等待任务分配后填写）

## task-empresas-pagination — 品牌列表页分页加载

- Status: ✅ 完成
- Files: 
  - `app/empresas/page.tsx`：全量加载 `getAllBrands()` → 分页加载 `queryBrands({ pageNum, pageSize })`，加载更多按钮
  - `lib/i18n/index.ts`：添加 `common.previous`、`common.next` 类型定义
  - `lib/i18n/translations/zh.json`、`es.json`：添加 previous/next 翻译
- Summary: 品牌列表页从全量加载改为分页加载（queryBrands），标题显示总数量 total，底部加载更多按钮

## task-online-offline — 上下架统一改造第二期+第三期

- Status: ✅ 完成
- Report: [task-online-offline/task_plan.md](task-online-offline/task_plan.md)
- Summary: API 层删除 deprecated 旧函数，三个页面 handleOnline/handleOffline 规范化（Admin 必填理由、统一 loadDetail()、Admin 上架入口已打开）

## task-b3-user-apply-page — 用户品牌认证申请入口页面

- Status: ✅ 开发完成，待审查
- Report: [task-b3-user-apply-page/task_plan.md](task-b3-user-apply-page/task_plan.md)
- Summary: 新建 BrandApplyMyPage.tsx（/brands/apply-my），四种状态展示 + 申请弹窗表单 + 撤回确认，60+ i18n key 三语言同步


## task-b3-fix-admin-pages — 修复品牌申请页面规范违规

- Status: ✅ 完成
- Summary: BrandApplyManagement + BrandApplyDetail 全面修复：静态 message→AppProvider、isAdmin→permissionList、40+ 硬编码→t()、50 新 i18n key 三语言同步、废弃 maxWidth 移除


## task-brand-management-refactor — 品牌管理页面三处改动

- Status: ✅ 完成，reviewer 审查通过，已修复所有反馈问题
- Files: routes.tsx / BrandDashboard.tsx / brand.ts / zh-CN.json / en.json / es-ES.json
- Summary: 菜单重构（ADMIN二级/BASE单入口）、Dashboard新增产品列表区块+流量数据占位、12个新i18n key三语言同步

## task-v1.3-brand-dashboard — v1.3 BrandDashboard 产品列表 + 流量占位 + reviewer 问题修复

- Status: ✅ 完成，reviewer 二次审查通过（report-round2.md）
- Commit: 228a3dd（2026-04-13）
- Files:
  - `src/api/brand.ts`：L244-246，BrandProductItem + apiGetBrandProducts，路径已对齐后端 `/brand/my-products`（Controller 确认）
  - `src/pages/BrandDashboard.tsx`：产品列表 Table（L104-142, L339-365）+ 流量数据占位（L367-380）；catch 里 setProductsError + Table locale.emptyText（L363）；L90 无硬编码兜底
  - `src/config/routes.tsx`：ADMIN 父节点虚拟路径 `/brands/_group`（L47），避免与 BASE `/brands` 路径冲突
  - `src/locales/zh-CN.json / en.json / es-ES.json`：brandDashboard.productList.*（7 key）+ brandDashboard.trafficData.*（2 key）+ common.refresh/loadFailed/noData 三语言全部就绪

### reviewer 4 个问题修复状态
- MEDIUM-1（接口路径）：`/brand/my-products` 已与后端 Controller 对齐 ✅
- MEDIUM-2（静默失败）：catch 设置 productsError=true，Table locale.emptyText 显示 t('common.loadFailed') ✅
- MEDIUM-3（路由重复）：ADMIN 父节点改为 `/brands/_group` 虚拟路径 ✅
- LOW-1（硬编码兜底）：移除 `|| '刷新'`，common.refresh 三语言均有 ✅

### 验证
- TypeScript 编译：通过（0 错误）

## task-v1.4-sticky-footer — ContentDetailLayout 底部操作栏统一改造（Task #4）

- Status: ✅ 完成，待 reviewer 审查
- Files:
  - `src/components/StickyFooterBar.tsx`：新建，左侧保存时间 + 右侧操作按钮，dayjs 相对时间每分钟刷新
  - `src/components/ContentDetailLayout.tsx`：新增 `footerBar?: ReactNode` prop，在 Spin 之后渲染
  - `src/pages/AgronomistEditor.tsx`：新增 lastSavedAt state，import StickyFooterBar，移除 Header onSave/onSaveAndPublish，传入 footerBar
  - `src/pages/StoreEditor.tsx`：同上，三处保存成功加 setLastSavedAt
  - `src/pages/BrandEditor.tsx`：同上，handleDraftSave/handleDetailAdminSave 加 setLastSavedAt
  - `src/pages/ProductEditor.tsx`：同上，detail 模式 ContentDetailLayout 接入 footerBar
  - `src/locales/zh-CN.json / en.json / es-ES.json`：新增 `common.neverSaved / lastSavedAt / justNow` 三语言
  - `docs/frontend/ui-components.md`：新增 StickyFooterBar 规范章节，标注「禁止在 Header 放保存按钮」

### 关键决策
- ProductEditor create/draft 模式保留原有按钮布局（未用 ContentDetailLayout），只对 detail 模式接入 footerBar
- `onSaveAndPublish` prop 保留（向后兼容），各 Editor 传空函数 `() => {}`，实际操作改为 StickyFooterBar extraActions
- dayjs relativeTime 按语言动态切换（zh-cn / es / en）

### 验证
- TypeScript 编译：通过（0 错误）
