# frontend-dev - 发现索引

> 纯索引——每个条目应简短（Status + Report 链接 + Summary）。

---

（等待任务分配后填写）

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

- Status: ✅ 完成，待 reviewer 审查
- Files: routes.tsx / BrandDashboard.tsx / brand.ts / zh-CN.json / en.json / es-ES.json
- Summary: 菜单重构（ADMIN二级/BASE单入口）、Dashboard新增产品列表区块+流量数据占位、12个新i18n key三语言同步；apiGetBrandProducts接口路径待backend-dev确认
