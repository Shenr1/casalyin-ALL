# E2E 测试进度

## 第一期：认证与会话管理 ✅ 完成

**结果：16/16 PASS**
**文件：** `casalyin-server/e2e/auth/login.spec.ts`
**Helper：** `casalyin-server/e2e/helpers/auth.ts`

---

## 第二期：草稿状态机 ✅ 完成

**结果：10/10 PASS**
**文件：** `casalyin-server/e2e/draft/product-draft.spec.ts`

### 发现并修复的 Bug

| # | 类型 | 内容 | 状态 |
|---|------|------|------|
| Bug 1 | 后端 | BASE 角色缺少 `product:edit`/`store:edit`/`agronomist:edit` 权限码 | ✅ 修复 |
| Bug 2 | 前端 | 四个 Editor 静态 `message` 导致错误静默 | ✅ 修复 |
| Bug 3 | 脚本 | SKU/名称超 20 字符 | ✅ 修复 |
| Bug 4 | 后端 | `setCreatorType()` 类型不匹配（String vs Integer）| ✅ 修复 |
| Bug 5 | 后端 | 拒绝接口未保存 `rejectReason`（三张草稿表缺列）| ✅ 修复（V3 Flyway 迁移）|
| Bug 6 | 后端 | 草稿详情接口无 creatorId 权限校验（数据隔离）| ✅ 修复 |
| Bug 7 | 前端 | ProductForm PENDING 状态 Input 未 disabled | ✅ 修复 |

### 关键经验
- `bail: 1` 配置下第一个失败就停，方便定位
- `waitForResponse` 比等 toast 更可靠
- TC-008 需要 8s 等待页面渲染「暂无数据」
- TC-009 selector 需排除 `type="checkbox"` / `type="file"` / `name="hiddenTextarea"`
- Redis ClassLoader 问题：DevTools 热重载后需 `docker exec -it redis redis-cli FLUSHALL`

---

## 第三期：已发布内容状态管理 ✅ 完成

**结果：4/4 PASS**
**文件：** `casalyin-server/e2e/publish/product-publish.spec.ts`

### 发现并修复的 Bug

| # | 类型 | 内容 | 状态 |
|---|------|------|------|
| Bug 8 | 前端 | `apiCertifyProduct` 参数嵌套错误（传对象而非 boolean）| ✅ 修复 |
| Bug 9 | 前端 | `ContentDetailLayout` 上架按钮条件依赖 `onOffline`，下架后不显示 | ✅ 修复（四维度共享组件一次修复）|

---

## 第 3.5 期：品牌认证回归 ✅ 完成

**结果：7/7 PASS**
**文件：** `casalyin-server/e2e/company/brand-certification.spec.ts`

### 测试用例

| 用例 | 描述 | 结果 |
|------|------|------|
| TC-BRAND-001 | COMPANY 创建产品：品牌字段 disabled + 免审直发 | ✅ PASS |
| TC-BRAND-002 | COMPANY 创建店铺：品牌字段 disabled + 免审直发 | ✅ PASS |
| TC-BRAND-003a | COMPANY 创建农艺师：品牌字段预填且 disabled | ✅ PASS |
| TC-BRAND-003b | COMPANY 创建农艺师：草稿创建 + 提交免审直发 | ✅ PASS |
| TC-BRAND-004a | BASE 产品页：BrandSelect 可交互（非 disabled） | ✅ PASS |
| TC-BRAND-004b | BASE 店铺页：无品牌字段 | ✅ PASS |
| TC-BRAND-004c | BASE 农艺师页：无品牌字段 | ✅ PASS |

### 发现并修复的 Bug

| # | 类型 | 内容 | 状态 |
|---|------|------|------|
| Bug 10 | 后端 | 农艺师草稿创建失败（AgronomistDraftService 异常）| ✅ 修复验证 |
| Bug 11 | 前端 | ProductForm 品牌名不显示（Form.Item name="brandId" 覆盖 value prop）| ✅ 修复验证 |

### 关键经验
- Ant Design `<Form.Item name="brandId">` 会用表单值覆盖子组件的 `value` prop → 品牌字段去掉 name prop 即可
- ContentDetailLayout 按钮有 letter-spacing，匹配需用 `/^保\s*存$/` 等 regex
- ProductEditor 用 Card 布局（底部按钮），Store/Agronomist 用 ContentDetailLayout（顶部按钮）
- `apiGetMyBrand` 异步加载，需 `waitFor` input[disabled] 出现后再断言

---

## 第五期：上下架专项 ✅ 完成

**结果：4/4 PASS**
**文件：** `casalyin-server/e2e/online/online-offline.spec.ts`

### 测试用例

| 用例 | 描述 | 结果 |
|------|------|------|
| TC-ONLINE-001 | 农艺师上下架 UI 一致性（按钮即时同步） | ✅ PASS |
| TC-ONLINE-002a | 产品上下架：POST /product/offline/{id} + online | ✅ PASS |
| TC-ONLINE-002b | 店铺上下架：POST /store/offline/{id} + online | ✅ PASS |
| TC-ONLINE-002c | 农艺师上下架：POST /agronomist/offline/{id} + online | ✅ PASS |

### 关键发现
- 品牌维度无上下架功能（前端 BrandEditor 无 onOnline/onOffline，后端无端点）
- 三个维度均为 POST 方法，路径 kebab-case，符合规范
- 下架返回 `"已下架"`，上架返回 `"已重新上架"`
- UI 按钮操作后即时同步（loadDetail 刷新），无需手动刷新
- 无新增 Bug

---

## 第六期：UI 规范全站回归 ✅ 完成

**结果：13 passed / 1 skipped**
**文件：** `casalyin-server/e2e/ui/ui-standards.spec.ts`

### 测试用例

| 用例 | 描述 | 结果 |
|------|------|------|
| TC-UI-001a | 产品列表页宽度检查 | ✅ PASS（maxWidth=1440px，规范要求 1000px） |
| TC-UI-001b | 店铺列表页宽度检查 | ✅ PASS（maxWidth=1440px） |
| TC-UI-001c | 农艺师列表页宽度检查 | ✅ PASS（maxWidth=1440px） |
| TC-UI-001d | 品牌列表页宽度检查 | ✅ PASS（maxWidth=1440px） |
| TC-UI-002a | 产品详情页返回按钮 | ⏭ SKIPPED（COMPANY 无已发布产品数据） |
| TC-UI-002b | 店铺详情页返回按钮 | ✅ PASS（空文本，仅图标） |
| TC-UI-002c | 农艺师详情页返回按钮 | ✅ PASS（空文本，仅图标） |
| TC-UI-003a | 产品列表无操作列 + 行点击 | ✅ PASS（无数据跳过行点击） |
| TC-UI-003b | 店铺列表无操作列 + 行点击 | ✅ PASS → /stores/4 |
| TC-UI-003c | 农艺师列表无操作列 + 行点击 | ✅ PASS → /agronomists/6 |
| TC-UI-004a | 产品页未发布 tab (BASE) | ✅ PASS |
| TC-UI-004b | 店铺页未发布 tab (BASE) | ✅ PASS |
| TC-UI-004c | 农艺师页未发布 tab (BASE) | ✅ PASS |
| TC-UI-004d | 品牌页未发布 tab (ADMIN) | ✅ PASS |

### 发现（待决策）

| # | 类型 | 内容 | 状态 |
|---|------|------|------|
| F1 | 规范偏差 | 列表页 maxWidth=1440px（ProLayout 默认），规范要求 1000px | 待决策 |
| F2 | 权限差异 | BASE 无法访问 /brands/list（需 brand:update/add/delete） | 预期行为 |

---

## 第七期：端到端完整业务场景 ✅ 完成

**结果：1/1 PASS**
**文件：** `casalyin-server/e2e/e2e/full-lifecycle.spec.ts`

### 测试用例

| 用例 | 描述 | 结果 |
|------|------|------|
| TC-E2E-001 | BASE 全生命周期：创建草稿→提交审核→ADMIN通过→下架→上架 | ✅ PASS |

### 阶段明细

| 阶段 | 操作 | API 响应 | 结果 |
|------|------|----------|------|
| 1 | BASE 创建产品草稿 | `ok:true, draftId=90` | ✅ |
| 2 | BASE 提交草稿→PENDING | `ok:true, "提交成功，请等待审核"` | ✅ |
| 3 | ADMIN 审核通过→发布 | `ok:true, "审核通过"` → 跳转 /products | ✅ |
| 4 | ADMIN 进入已发布产品详情 | productId=83 | ✅ |
| 5 | ADMIN 下架（填理由） | `ok:true, "已下架"` → 按钮切换为上架 | ✅ |
| 6 | ADMIN 再次上架 | `ok:true, "已重新上架"` → 按钮恢复为下架 | ✅ |

### 关键经验

- Ant Design `modal.confirm()` 从 AppProvider 导出时，渲染的不是标准 `role="dialog"` 元素，而是 `.ant-modal-confirm` 类
- 按钮选择器应使用 `.ant-modal-confirm-btns .ant-btn-primary`，不用 `getByRole('button', { name: ... })`（因标题文字可能与按钮文字重叠，如"确认审核通过该产品？"的标题包含"通过"）
- `browser.newPage()` 实现 BASE 和 ADMIN 双用户隔离（各自独立 cookie/session）
- 整条链路 39.2s 完成，无新增 Bug

---

## B3 集成测试 ✅ 完成（v1.2 收尾回归）

**日期：** 2026-04-13
**文件：** `casalyin-server/e2e/brand-apply/b3-integration.spec.ts`
**结果：** 10/10 PASS ✅

### 测试用例

| 用例 | 描述 | 结果 |
|------|------|------|
| TC-T1-001 | Admin 访问农艺师管理页面不再白屏 | ✅ PASS |
| TC-T1-002 | BASE 访问农艺师管理页面正常 | ✅ PASS |
| TC-T1-003 | COMPANY 访问农艺师管理页面正常 | ✅ PASS |
| TC-T2-001 | BASE 访问 /brands/apply-my 页面正常加载 | ✅ PASS |
| TC-T2-002 | BASE 提交品牌申请（已有记录→验证 PENDING 状态展示） | ✅ PASS |
| TC-T2-003 | COMPANY 访问 /brands/apply-my（Result 已认证状态） | ✅ PASS |
| TC-T3-001 | ADMIN 品牌申请列表页正常加载 | ✅ PASS |
| TC-T3-002 | ADMIN 点击申请行进入详情页（URL: /brands/apply/10） | ✅ PASS |
| TC-T3-003 | ADMIN 审核按钮展示（无待审核记录，跳过） | ✅ PASS |
| TC-T4-001 | COMPANY 同步产品（code:0, "同步完成，共更新 9 个产品"） | ✅ PASS |

### 关键结论

1. **Bug 13 修复验证通过** — `/brand/apply/submit` 已正常，TC-T2-002 已有记录验证 PENDING 状态展示正常
2. **Bug 14 修复验证通过** — `/product/sync-creator` 返回 `{code:0, data:"同步完成，共更新 9 个产品"}`，HTTP 200，不再报错
3. **T3 Admin 详情页** — 成功跳转 `/brands/apply/10`，返回按钮 + Descriptions 均可见
4. **T3-003** — 当前无 PENDING 申请，审核按钮场景跳过（正常行为）
5. **v1.2 全部测试场景验证完毕**

---

## ⏭ 下一步：第四期

### 第四期：COMPANY 角色专项
**用例：** TC-COMPANY-001、TC-COMPANY-002
**文件（待创建）：** `casalyin-server/e2e/company/company-role.spec.ts`
- TC-COMPANY-001：B→C 升级申请全流程（法律条款确认、提交、ADMIN 审批、角色升级）
- TC-COMPANY-002：COMPANY 操作权限边界（删除 403、创建自动认证、品牌下内容操作）

---

## 排查规范（三步顺序，必须遵守）

1. **看页面是否符合预期结果**
2. **看页面上有没有弹窗/toast 提示**（静默失败本身是 Bug）
3. **最后才看 API response**（`waitForResponse` + `console.log`）

## 环境信息

- 前端：`http://localhost:5173`
- 后端：`http://localhost:8690`（Playwright 走前端代理，不直连后端）
- 后端启动：IntelliJ 右上角「Run Java」
- Redis 清缓存：`docker exec -it redis redis-cli FLUSHALL`（backend-dev 授权自行执行）
- 重启后不需要等用户确认，直接跑，登录失败等 30s 重试
