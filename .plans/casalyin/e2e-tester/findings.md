# e2e-tester - 测试发现索引

> 纯索引——每个条目简短（Status + Report 链接 + Pass rate + Summary）。

---

## [BUG] 2026-04-11 — AgronomistManagement 引用不存在的导出，导致整个 React App 崩溃

**严重度：CRITICAL**

**来源：** 上下架 E2E 测试任务（online-offline.spec.ts）

**复现步骤：**
1. 启动 Playwright 测试，访问任意页面（包括 Admin 登录页）
2. 浏览器控制台报错：`The requested module '/src/api/agronomist.ts' does not provide an export named 'apiAdminToggleAgronomistDisabled'`
3. React 应用初始化失败，`#root` 始终为空，所有测试均超时

**根因：**
- `AgronomistManagement.tsx:15` import 了 `apiAdminToggleAgronomistDisabled`
- 该函数已在上下架改造中被删除/重命名，`agronomist.ts` 现在只有 `apiOnlineAgronomist` / `apiOfflineAgronomist`
- ES module 静态导入失败 → React 无法初始化 → 所有页面白屏

**影响范围：** 所有 E2E 测试（12/12 失败），整个前端 App 无法运行

**文件位置：**
- `casalyin-server/src/pages/AgronomistManagement.tsx` 第 15 行（import）、第 216 行（调用）
- `casalyin-server/src/api/agronomist.ts`（无 `apiAdminToggleAgronomistDisabled` 导出）

**建议修复：**
在 `AgronomistManagement.tsx` 中，将 `handleToggleDisabled` 函数改为调用 `apiOnlineAgronomist` / `apiOfflineAgronomist`，并根据 `record.disabledFlag` 分支处理。注意 Admin 下架需要填写原因（参考 AgronomistEditor.tsx 里的实现）。

**Pass rate：** 0/12 (0%)

**状态：** ✅ 已修复（2026-04-13 B3 集成测试验证通过）

---

## [BUG] 2026-04-13 — 品牌申请提交 API 返回系统错误 (code:10001)

**严重度：HIGH**

**来源：** B3 集成测试（b3-integration.spec.ts TC-T2-002）

**复现步骤：**
1. BASE 用户登录（123456@qq.com）
2. 访问 `/brands/apply-my` → 点击「申请认证」
3. 弹窗中通过 BrandSelect 输入新品牌名（触发「+ 新增」）
4. 填写申请说明 → 点击「提交申请」
5. POST `/brand/apply/submit` 返回 `{code:10001, msg:"系统似乎出现了点小问题"}`

**根因：** 后端 BrandApplyController 或 Service 层异常，具体需查看后端日志。

**影响范围：** BASE 用户无法提交品牌认证申请

**建议：** 交 backend-dev 查看后端日志，定位 `/brand/apply/submit` 的异常堆栈

**状态：** ✅ 已修复（2026-04-13 v1.2 回归验证通过）

---

## [BUG] 2026-04-13 — 同步产品归属 API 返回系统错误 (code:10001)

**严重度：HIGH**

**来源：** B3 集成测试（b3-integration.spec.ts TC-T4-001）

**复现步骤：**
1. COMPANY 用户登录（123123@qq.com）
2. 访问 `/products` → 点击「操作」下拉 → 选择「同步产品归属」
3. 确认弹窗点击「确定」
4. POST `/product/sync-creator` 返回 `{code:10001, msg:"系统似乎出现了点小问题"}`

**根因：** 后端 ProductController#syncCreator 异常。注意：HTTP 状态码是 200（不是 403），说明 Bug 12 的权限修复已生效（product:edit 权限码修复成功），但接口内部逻辑有异常。

**影响范围：** COMPANY 用户无法同步产品归属

**建议：** 交 backend-dev 查看后端日志，定位 `/product/sync-creator` 的异常堆栈

**状态：** ✅ 已修复（2026-04-13 v1.2 回归验证通过，返回 `code:0, "同步完成，共更新 9 个产品"`）

---

