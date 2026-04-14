# 交接文档 — 2026-04-13

> 用途：/clear 后恢复团队状态用

---

## 当前遗留 Bug（需立即处理）

### Bug 13：POST /brand/apply/submit 返回系统错误
- **严重度**：HIGH
- **现象**：前端表单提交正常，后端返回 `{code:10001, msg:"系统似乎出现了点小问题"}`
- **接口**：`POST /brand/apply/submit`
- **Service**：`BrandApplyService.submitApply()`（约第84行）
- **排查方向**：
  1. `(RequestUser) SmartRequestUtil.getRequestUser()` 强转是否安全
  2. `brandApplyDao.insert(entity)` 字段与 DB 列是否匹配
  3. MyBatis Plus 自动填充与手动 `setCreateTime()` 是否冲突
  4. **最需要的**：看 IntelliJ 后端控制台实际异常堆栈

### Bug 14：POST /product/sync-creator 返回系统错误
- **严重度**：HIGH
- **现象**：HTTP 200 但 `{code:10001}`，不再是 403（Bug 12 权限码已修复验证通过）
- **接口**：`POST /product/sync-creator`
- **Service**：`ProductService.syncCreatorToBrandUser()`（约第941行）
- **排查方向**：
  1. `productDao.updateOwnerUserIdByBrandId()` SQL 是否正确
  2. 测试 COMPANY 账号（123123@qq.com）是否有 status=1 的 brand_apply 记录
  3. **最需要的**：看 IntelliJ 后端控制台实际异常堆栈

---

## B3 测试结果概览（e2e 8/10 PASS）

| 测试项 | 结果 |
|--------|------|
| T1 AgronomistManagement 白屏修复 | ✅ 3/3 PASS |
| T2 /brands/apply-my 用户申请页 | ⚠️ 2/3（Bug 13 导致提交失败） |
| T3 品牌申请管理端（Admin） | ✅ 3/3 PASS（无数据，部分跳过） |
| T4 /product/sync-creator 同步产品 | ❌ FAIL（Bug 14） |

**Bug 13 修复后**：T2 需回归（提交申请全流程）
**Bug 14 修复后**：T4 需回归（同步产品功能）
**T3 管理端**：需在 T2 修复后补充有数据的审核场景回归

---

## 已完成工作汇总（本轮）

| 任务 | 状态 |
|------|------|
| B1 产品敏感词检测 | ✅ 确认已存在，无需改动 |
| B2 RoleUtils 工具类 | ✅ 完成 |
| B3-1 后端接口梳理 | ✅ 完成 |
| B3-2 BrandApplyManagement/Detail 规范修复 | ✅ reviewer [OK] |
| B3-3 /brands/apply-my 用户申请入口页 | ✅ reviewer [OK] |
| B-品牌3 同步产品归属 | ✅ 完成 |
| Bug 12 product:edit 权限码 | ✅ 修复并验证（不再 403） |
| 白屏热修复（AgronomistManagement） | ✅ 修复并验证 |
| BrandApplyController 路径参数 {id} | ✅ 修复 |
| ProductController 路径参数 {id} | ✅ 修复 |

---

## Backlog 剩余（未开始）

| # | 内容 | 优先级 |
|---|------|--------|
| B4 | i18n 补全：productForm.brand / storeForm.fields.brand 缺 en/es-ES | 低 |
| D-论坛1 | Discourse 关联第一期（后台 discourse_tag 字段） | 待排期 |
| D-论坛2 | Discourse 关联第二期（Headless 前台展示） | 待排期 |

详细 Discourse 计划见：`.plans/casalyin/discourse-integration/plan.md`

---

## 重启团队后第一步

1. 读本文件恢复状态
2. 让 backend-dev 查 IntelliJ 控制台日志修复 Bug 13 和 Bug 14
3. Bug 修复后通知 e2e-tester 回归 T2 和 T4
