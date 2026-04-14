# B3-1 后端：梳理品牌申请接口完整性

> 状态: ✅ 已完成
> 日期: 2026-04-12

## 检查结论

### ✅ 确认项（无问题）

| # | 检查项 | 结果 |
|---|--------|------|
| 1 | BrandApplyForm 字段完整性 | ✅ 包含 brandId/companyName/applyReason/licenseUrl，校验注解完善 |
| 2 | BrandApplyVO 返回字段 | ✅ 包含完整展示所需字段，前端 TS 类型完全对齐 |
| 3 | cancel 的 applyId 范围校验 | ✅ Service 层有 userId 校验（只能撤自己的） |
| 4 | 权限码 brand:apply 注册 | ✅ 已在 BASE 和 COMPANY 权限列表中注册 |

### ⚠️ 修复项

| # | 问题 | 修复 |
|---|------|------|
| 1 | Controller 路径参数用了 {applyId}/{userId}，违反规范 | 全部改为 {id}（5处） |

## 修复的代码变更

文件：BrandApplyController.java

变更：路径参数 `{applyId}` → `{id}`（3处），`{userId}` → `{id}`（2处）

涉及接口：
- `/brand/apply/approve/{id}` — approve()
- `/brand/apply/cancel/{id}` — cancelApply()
- `/brand/apply/reject/{id}` — reject()
- `/admin/brand-apply/subscription/{id}` — getSubscription()
- `/admin/brand-apply/revoke/{id}` — revokeCompany()

**注意**：路径变更是纯路径参数重命名（{applyId}→{id}、{userId}→{id}），URL 路径结构不变，前端调用不受影响（前端只关心 URL 路径，不关心参数名）。

## 接口完整性总结

| 接口 | 路径 | 方法 | 权限码 | 备注 |
|------|------|------|--------|------|
| 提交申请 | /brand/apply/submit | POST | 仅需登录 | 防重复校验 ✅ |
| 查询我的申请 | /brand/apply/my | GET | 仅需登录 | 按时间倒序 ✅ |
| 撤回申请 | /brand/apply/cancel/{id} | POST | 仅需登录 | userId 校验 ✅ |
| 我的订阅 | /brand/apply/my-subscription | GET | brand:apply | COMPANY 自查 ✅ |
| 全部列表 | /brand/apply/list | GET | brand:audit | Admin ✅ |
| 待审核列表 | /brand/apply/pending | GET | brand:audit | Admin ✅ |
| 待审核数量 | /brand/apply/pending/count | GET | brand:audit | Dashboard ✅ |
| 审核通过 | /brand/apply/approve/{id} | POST | brand:audit | Admin ✅ |
| 审核拒绝 | /brand/apply/reject/{id} | POST | brand:audit | Admin ✅ |
| 查询订阅 | /admin/brand-apply/subscription/{id} | GET | brand:audit | Admin ✅ |
| 更新订阅 | /admin/brand-apply/subscription/update | POST | brand:audit | Admin ✅ |
| 撤销认证 | /admin/brand-apply/revoke/{id} | POST | brand:audit | Admin ✅ |
