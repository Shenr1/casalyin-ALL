# Task: B3-3 用户品牌认证申请入口页面

> Status: in_progress
> Started: 2026-04-12

## 目标

新建 `/brands/apply-my` 路由页面，供 BASE/COMPANY 用户查看品牌认证申请状态和提交申请。

## 后端接口（已确认）

| 功能 | 方法 | 路径 | 权限 |
|------|------|------|------|
| 提交申请 | POST | /brand/apply/submit | 登录即可 |
| 查我的申请 | GET | /brand/apply/my | 登录即可 |
| 撤回申请 | POST | /brand/apply/cancel/{applyId} | 登录即可 |
| 查我的订阅 | GET | /brand/apply/my-subscription | brand:apply |

### BrandApplyForm 字段
- brandId?: number（认领品牌时传）
- companyName: string（必填）
- applyReason: string（必填）
- licenseUrl?: string（选填）

### BrandApplyVO 关键字段
- applyId, status(0-待审核/1-已批准/2-已拒绝/3-已撤销)
- companyName, applyReason, licenseUrl
- rejectReason, createTime, auditTime
- subscriptionExpiredAt, subscriptionStatus, subscriptionFrozen

## 开发清单

- [ ] 1. 创建 BrandApplyMyPage.tsx 页面组件
  - 四种状态展示：无申请 / PENDING / APPROVED / REJECTED
  - 申请表单（弹窗 Modal）
  - 撤回确认
- [ ] 2. 注册路由 `/brands/apply-my`
  - routes/index.tsx
  - config/routes.tsx（菜单项）
- [ ] 3. i18n：新增 brandApplyMy.* 命名空间
  - zh-CN.json, en.json, es-ES.json 三语言同步
- [ ] 4. 更新路由清单文档 docs/frontend/routing.md
- [ ] 5. 更新工作目录 findings.md + progress.md
- [ ] 6. 请求 reviewer 审查

## 待确认

- ContentDetailLayout 是否适合此页面（已上报 team-lead）
