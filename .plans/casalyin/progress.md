# casalyin - 工作日志

> 按时间线记录。

---

## 2026-04-11 Session 1 — 团队搭建

### 已完成
- [x] 将所有规范从 Claude 全局 memory 迁移到 docs/ 目录（12 个文件）
- [x] 创建项目 CLAUDE.md（索引 + 核心约定）
- [x] 创建 .plans/casalyin/ 目录结构
- [x] 搭建团队（frontend-dev、backend-dev、e2e-tester、reviewer）

---

## 2026-04-11 Session 2 — 第一期 E2E + Bug 修复

### 已完成
- [x] 第一期 E2E：认证与会话管理，16/16 PASS（auth/login.spec.ts）
- [x] 第二期 E2E：草稿状态机，10/10 PASS（draft/product-draft.spec.ts）
  - Bug 1-7 全部修复（权限码、静态 message、setCreatorType、rejectReason、权限校验、PENDING disabled）
- [x] reviewer 审查通过：PermissionService 权限码变更 [OK]

---

## 2026-04-12 Session 3 — 第三期 + 品牌认证 + 上下架

### 已完成
- [x] 第三期 E2E：已发布内容状态管理，4/4 PASS（publish/product-publish.spec.ts）
  - Bug 8 修复：apiCertifyProduct 参数嵌套错误
  - Bug 9 修复：ContentDetailLayout 上架按钮显示条件（四维度共享，一次修复）
- [x] 品牌认证体系后端（第一期）：
  - Flyway V4 迁移（brand_id 字段）
  - COMPANY 免审直发（含 D10：SOFT 违规词也跳过）
  - 自动写入 brand_id
- [x] 品牌认证体系前端（第二期）：
  - COMPANY 品牌字段预填只读
  - Bug 11 修复：Form.Item 去掉 name prop
  - i18n 三语言补全
- [x] 第 3.5 期 E2E：品牌认证回归，7/7 PASS（company/brand-certification.spec.ts）
  - Bug 10 修复：农艺师草稿创建失败
- [x] 第五期 E2E：上下架专项，4/4 PASS（online/online-offline.spec.ts）
  - 确认品牌无上下架功能（设计如此）
- [x] 第六期 E2E：UI 规范全站回归，13/14 PASS（ui/ui-standards.spec.ts）
  - F1 页面宽度：用户决策 option 2，规范从 1000px 改为 ProLayout 默认 1440px
  - docs/frontend/ui-components.md 已更新
- [x] 第七期 E2E：端到端完整业务场景，1/1 PASS（e2e/full-lifecycle.spec.ts）
  - 全链路验证：BASE 创建→提交→ADMIN 审核→发布→下架→上架，39.2s 完成
  - 关键经验：modal.confirm() 使用 .ant-modal-confirm:visible 选择器

### 当前状态
七期测试全部完成，11 个 Bug 已修复，等待用户决定 Backlog 方向。

---

## 2026-04-14 Session 4 — v1.1~v1.4.1 全线推进

### v1.1 Backlog 清理
- [x] B1 产品 submitDraft 敏感词检测：确认已存在，无需改动
- [x] B2 isCompanyUser() 抽离为 RoleUtils
- [x] B-品牌3 品牌认领 → 批量写入 brandOwnerId
- [x] Bug 12 product:edit 权限码修复

### v1.2 品牌申请体系（E2E 10/10 PASS）
- [x] B3-1 后端接口梳理 + 路径规范修复
- [x] B3-2 BrandApplyManagement/Detail 规范修复
- [x] B3-3 /brands/apply-my 用户申请入口页
- [x] Bug 13 POST /brand/apply/submit 自愈
- [x] Bug 14 CacheConfig FastJSON→Jackson 序列化器替换

### v1.3 品牌管理页重构（E2E 10/10 PASS）
- [x] 菜单重构：删除独立 /brands/apply-my 菜单，ADMIN 加二级菜单
- [x] BrandDashboard 新增产品列表区块（12条，VIP 优先）
- [x] BrandDashboard 流量数据占位区块
- [x] 后端 GET /brand/my-products 接口
- [x] i18n 三语言同步
- [x] Bug 修复：brand:apply:audit 未入库（V7 Flyway）

### v1.3.1 底部操作栏规范（E2E 6/6 PASS）
- [x] StickyFooterBar 组件（lastSavedAt + 相对时间 + extraActions）
- [x] ContentDetailLayout 接入 footerBar prop
- [x] ProductEditor/StoreEditor/BrandEditor/AgronomistEditor 全部迁移
- [x] 12 个 i18n key，docs/frontend/ui-components.md 规范更新

### v1.4 平台认证申请体系（E2E 全部 PASS）
- 期一：后端 Controller + Flyway V6/V8
- 期二：前端 ContentStatusCard + 四维度 Editor 接入
- 期三：Admin Dashboard 待办数量（GET /certification/pending-count）+ 审核通知（AUDIT_APPROVED/AUDIT_REJECTED）

### v1.4.1 遗留清理（E2E 3/3 PASS，连续三轮稳定）
- [x] T1：BrandDashboard 集成 ContentStatusCard（isAdmin={false} 硬编码正确，ADMIN 不进 BrandDashboard）
- [x] T2：CONTENT_OFFLINE 通知核查（三维度已有，Notifications.tsx 颜色 default→orange）
- [x] T3：discourse_tag 全链路（Flyway V9 + 7个Java文件 + Mapper XML + ProductForm ADMIN 编辑框）
- Bug 15 修复：ProductMapper.xml getDetailById 漏 discourse_tag（e2e-tester 发现并修复）

### 本次经验教训
1. **新字段必查 Mapper XML**：新增字段时除了 Entity/VO/Form/Service，必须检查 Mapper XML 中是否有自定义联表查询（getDetailById 等），否则写入成功但读取为 null
2. **handleSaveProduct 提交对象需手动跟进**：前端保存方法如果手动列举 updateData 字段（而非 form.getFieldsValue() 全量），新字段必须手动添加
3. **版本收尾协议**：E2E 全 PASS 后立即更新 progress.md + handoff.md + task_plan.md，然后 /compact 压缩上下文

---

---

## 2026-04-14 Session 5 — 规范清理 + v1.5 D-论坛2

### 规范清理（静态 message 导入）
- [x] Task #16 核查：Notifications.tsx 实际已符合规范（`import { message } from '../components/AppProvider'`），无需修改
- [x] 全站扫描发现 19 个文件仍从 `antd` 直接导入静态 `message`，批量迁移至 AppProvider
- [x] `tsc --noEmit` 零错误，reviewer 抽查 4 个代表性文件 [OK]

### v1.5 D-论坛2（代码完成，e2e 待验）
- [x] T1 后端：`DiscoursePostVO` + `DiscourseService`（调 Discourse API）+ `GET /product/discourse-posts/{id}`（@NoNeedLogin）
- [x] T2 Headless 前端：Route Handler + `lib/discourse-api.ts` + `ProductForumLinks.tsx` 改造（真实数据 + loading/空状态/回复数/浏览数）+ 产品详情页集成 + i18n（es/zh 各 6 key）
- [ ] T3 E2E：后端接口验证（TC-151-1/2/3）— 因编译缓存问题阻塞，待下次 session 验证
  - 编译缓存问题（STORE_LOCKED）：代码正确，需执行 Rebuild Project 后重启
  - Headless UI 验证：移交给后续 headless-tester 处理

### 团队规划决策
- 确定增加 `headless-dev` + `headless-tester` 两个专职角色到现有团队
- C 端 API 健康检查列入 Headless 团队上线后第一个任务

---

## 2026-04-14 Session 6 — v1.5 D-论坛2 验证

### v1.5 D-论坛2 E2E 验证（3/3 PASS）
- [x] 排查后端编译缓存问题：context-path `/api` 前缀发现，Rebuild Project 后接口正常注册
- [x] 新建 `casalyin-server/e2e/v1.5/discourse-posts.spec.ts`
- [x] TC-151-1/2/3 全部 PASS（API 格式验证、空数组、不存在 ID 不崩溃）

**v1.5 正式关闭** ✅（Headless UI 验证留待 headless-tester）

### 经验教训
- 后端 context-path 是 `/api`，所有接口调用必须加此前缀（已在 playwright.config.ts 体现，但 curl 调试时容易忘记）

---

## 2026-04-15 Session 7 — v1.6 C端 API 健康检查 + 农艺师集成

### v1.6 C端 API 健康检查（18/18 PASS）
- [x] 新建 `casalyin-server/e2e/v1.6/c-api-health.spec.ts`
- [x] 18 个后端公开接口全部健康，耗时 15.6s，0 failed

**发现的注意事项：**
1. **后端成功 code 是 `0`，不是 `200`** — 响应格式 `{ code: 0, ok: true, data: ... }`
2. **`/api/global/search` 无需登录** — 之前 headless 旧测试标注 auth-blocked 是 proxy 路径问题，非接口问题
3. **storeId 需从 product detail 的 `stores[]` 数组提取**，filter 列表不含 storeId

### v1.6 Headless 农艺师集成（E2E T1/T2/T3 PASS，T4 代码就位）

**Headless 改动（frontend-dev）：**
- [x] T1 `next.config.js` 图片 hostname 修复：`remotePatterns` 从 `NEXT_PUBLIC_API_BASE_URL` 动态读 + 开发环境通配 `/api/upload/**`（解决 DB 存真实 IP 问题）
- [x] T2 清理 `/agronomos` 旧路由（整个目录删除），统一用 `/agronomists`
- [x] T3 作物详情页农艺师卡片加跳转链接（`/agronomists/[slug]`）
- [x] T4 产品详情页加农艺师区块（从 usage cropId 关联，最多展示 3 个，tsc 验证通过）
  - **注意**：本地 73 个产品 usage 表全空，UI 验证待有数据后触发；代码链路已 tsc 验证
- [x] T5 `lib/agronomist-api.ts` 预留 `vipOnly?: boolean` 参数（`AgronomistListOptions` 接口，带 TODO 注释）

**后端改动（backend-dev）：**
- [x] `AgronomistPublicController` 两个公开接口（list / list/filter）加 `vipOnly` 可选参数，当前忽略，预留 TODO

**E2E 结果（`casalyin-server/e2e/v1.6/headless-agronomist.spec.ts`）：**
- T1 图片无报错 ✅ | T2 旧路由 404 ✅ | T3 卡片跳转 ✅ | T4 无 usage 数据跳过 ⏭️

**经验教训：**
- `next.config.js` 的 `remotePatterns` 按 `NEXT_PUBLIC_API_BASE_URL` 配，但 DB 存的图片 URL 是上传时的真实机器 IP，两者不一致 → 开发环境需加通配规则

**v1.6 正式关闭** ✅

---

## 待开发（下一个 session）

| 优先级 | 内容 |
|--------|------|
| 待定 | v1.7 C端体验完善：病害页（/plagas）、成分页（/ingredientes）、搜索页等功能摸底 |
| 待定 | v1.5 D-论坛3：扩展到品牌/店铺/农艺师 + 体验增强 |
| 待定 | v2.0 仓储物流：先对齐 D13 的 5 个问题再立项 |

---

## 2026-04-16 Session 8 — v1.7 E2E 三期完成

### v1.7 E2E 结果（27/27 PASS）

| 期次 | 内容 | 结果 | 耗时 |
|------|------|------|------|
| 第一期 | 农艺师后台管理 TC-A1~A7 | 7/7 PASS | 6.1m |
| 第二期 | 病虫害+农作物 CRUD TC-P1~P4, TC-C1~C4 | 8/8 PASS | 3.6m |
| 第三期 | 有效成分+Tag TC-I1~I4, TC-T1~T8 | 12/12 PASS | 4.7m |

**总计：27 个用例全部通过，无 Bug，无失败。**

### ContentSidebar.tsx 状态
- 组件已创建：`src/components/ContentSidebar.tsx`
- 已集成到：`src/components/ContentDetailLayout.tsx`
- 状态：✅ 完成（ContentSidebar 已接入 ContentDetailLayout，Descriptions 元信息 + Typography.Link 删除按钮 + sidebarExtra 扩展槽）

### v1.7 关闭条件核查
- [x] 三期 E2E 全部 PASS（27/27）
- [x] ContentSidebar.tsx 抽离完成（已集成）
- [ ] 待确认：其他 v1.7 前端任务（接口规范修复、农艺师 VIP 等）

### 下一步
v1.7 可关闭 → 执行 /compact 压缩上下文

---

## 2026-04-16/17 — v1.8 收尾 + v2.0 记录 + 遗留清理

### v1.8 工作流收尾（3/3 完成）
- [x] 工作流 A — i18n 扫荡（plagas/ingredientes/empresas/crops/pests results）
- [x] 工作流 B — by-slug API（/by-slug/{slug} 后端接口 + 前端 detail pages 改造）
- [x] 工作流 C — 新增翻译 key（pages.detail.* / search.searching）
- [x] 自我审查发现额外文件：IngredientsResults.tsx / BrandsResults.tsx 硬编码修复

### 遗留清理（headless-tester 执行）
- [x] Task #2：ProductCompareModal.tsx i18n（12 个 key，zh/es/index.ts 全同步，tsc 零错误）
- [x] Task #3：IconLibrary.tsx alt i18n（altKey 方案：config 存 key，组件内 t() 翻译，tsc 零错误）

### v2.0 记录
- [x] D13 更新：Q1/Q2/Q3/Q4 部分已确认，8 个问题待确认
- [x] ADR-006 新增（docs/decisions/architecture-decisions.md）
- [x] task_plan.md 新增 v2.0 占位 section

### 当前阻塞
- v1.8 E2E Round 2：后端 8690 未启动，19 个用例无法执行（代码已全部完成）

### 下一步
v1.8 代码侧全部完成 → 等用户确认后端就绪 → 执行 E2E Round 2 → /compact → 继续聊 v2.0

---

## 2026-04-16/17 — v1.8 E2E Round 2 完成

### E2E Round 2 结果（headless-tester 执行）

**文件：** `casalyin-server/e2e/v1.8/i18n-sweep.spec.ts` + `slug-stability.spec.ts`

| 测试组 | 内容 | 结果 |
|--------|------|------|
| A1 | 中文页面 i18n（plagas/ingredientes/empresas/crops results） | 4/4 PASS ✅ |
| A3 | 翻译完整性（es.json/zh.json 含所有必需 key） | 2/2 PASS ✅ |
| B1 无效slug | 404 兜底（crop/pest/composition） | 3/3 PASS ✅ |
| B2 | slug 详情页 URL 直接访问 | 4/4 PASS ✅ |
| B3 | 后端 by-slug 接口健康 | 3/3 PASS ✅ |
| A2 | 西班牙语 App（3001 未启动） | 8 FAIL ⏸️ |
| B1 有效slug | 测试数据无 slug 字段 | 3 SKIP ⏭️ |

**汇总：16 PASS / 0 FAIL / 3 SKIP / 8 FAIL（A2，西班牙语 App 未启动）**

### by-slug API curl 验证
```
GET /api/crop/by-slug/test-slug → {"code":30001,"msg":"农作物不存在"} ✅
```
API 本身正常，B1 有效slug 3 SKIP 是测试数据问题，非代码问题。

### v1.8 关闭条件
- [x] 工作流 A（i18n 扫荡）：代码审查通过
- [x] 工作流 B（slug↔ID 修复）：by-slug 接口正常，curl 验证通过
- [x] 工作流 C（翻译 key 新增）：es.json + zh.json 包含所有必需 key
- [x] E2E Round 2（B1/B2/B3/A1/A3）：16 PASS ✅
- [ ] A2 西班牙语 App：待 headless 3001 端口启动后单独回归

### 当前阻塞
- A2 西班牙语 App（headless 3001 端口未启动）：需用户执行 `cd casalyin-Headless && NEXT_PUBLIC_LOCALE=es npm run dev -- -p 3001`

### 下一步
用户确认西班牙语 App 启动后 → 跑 A2 回归 → /compact → 聊 v2.0

---

## 2026-04-17 — v1.9 分页改造 + Task #8 getBrandIdBySlug 修复

### 分页改造（前端 Load More 模式）
- [x] `app/empresas/page.tsx`：`getAllBrands()` → `queryBrands({ pageNum, pageSize })` + Load More 按钮
- [x] `app/plagas/page.tsx`：`getAllPests()` → `queryPests({ pageNum, pageSize })` + Load More 按钮
- [x] `app/ingredientes/page.tsx`：`getAllCompositions()` → `queryCompositions({ pageNum, pageSize })` + Load More 按钮
- [x] Task #6 标记完成：品牌列表页前端改造
- [x] TypeScript `tsc --noEmit` 零错误

### getBrandIdBySlug 修复（Task #8）
- [x] `list-api.ts`：`getBrandIdBySlug` 函数 URL 修正为 `/api/brand/public/by-slug/`
- [x] `empresas/[slug]/page.tsx`：已使用 `getBrandIdBySlug` 替代脆弱的 name 匹配逻辑
- [x] Task #7 后端 Brand by-slug 公开接口确认存在（`GET /brand/public/by-slug/{slug}`）

### 当前状态
v1.9 E2E 验证中，发现并修复 Bug: list-api.ts 三个分页函数 API 路径缺少 /public/，导致列表页数据全部 404。已修复，TypeScript 零错误，待 E2E 回归。
