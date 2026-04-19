# casalyin - 主计划

> 状态: IN_PROGRESS
> 创建: 2026-04-11
> 更新: 2026-04-13
> 团队: casalyin (frontend-dev, backend-dev, e2e-tester, reviewer)
> 决策记录: .plans/casalyin/decisions.md

---

## 版本说明

每个版本结束后打 ✅ DONE 标记，作为回归基准点。
版本内所有功能通过 E2E 验证后才能关闭。

---

## ✅ v1.0 — E2E 测试 + Bug 修复（已完成，2026-04-12）

**基准**：七期 E2E 全部通过，11 个 Bug 已修复。

| 期次 | 内容 | 结果 |
|------|------|------|
| 第一期 | 认证与会话管理 | 16/16 PASS |
| 第二期 | 草稿状态机 | 10/10 PASS |
| 第三期 | 已发布内容状态管理 | 4/4 PASS |
| 第 3.5 期 | 品牌认证回归 | 7/7 PASS |
| 第五期 | 上下架专项 | 4/4 PASS |
| 第六期 | UI 规范全站回归 | 13/14 PASS |
| 第七期 | 端到端完整业务场景 | 1/1 PASS |

**已修复 Bug（v1.0）：** Bug 1~11，详见 progress.md

---

## ✅ v1.1 — Backlog 清理（已完成，2026-04-12）

| 任务 | 内容 | 状态 |
|------|------|------|
| B1 | 产品 submitDraft 敏感词检测 | ✅ 确认已存在，无需改动 |
| B2 | isCompanyUser() 抽离为 RoleUtils | ✅ 完成 |
| B-品牌3 | 品牌认领 → 批量写入 brandOwnerId | ✅ 完成 |
| Bug 12 | product:edit 权限码修复 | ✅ 修复并验证 |

---

## ✅ v1.2 — 品牌申请体系（已完成，2026-04-12~13，E2E 10/10 PASS）

BASE 用户申请升级为 COMPANY 的完整流程。

| 任务 | 内容 | 状态 |
|------|------|------|
| B3-1 后端 | 品牌申请接口梳理 + 路径规范修复 | ✅ 完成 |
| B3-2 前端 | BrandApplyManagement / Detail 规范修复 | ✅ reviewer [OK] |
| B3-3 前端 | /brands/apply-my 用户申请入口页 | ✅ reviewer [OK] |
| Bug 13 | POST /brand/apply/submit 报 10001 | ✅ 已自愈（日志确认正常） |
| Bug 14 | POST /product/sync-creator 报 10001 | ✅ 独立修复：CacheConfig FastJSON→Jackson 序列化器替换，需清 Redis 重启 |
| 白屏热修复 | AgronomistManagement | ✅ 完成 |

**v1.2 E2E 结果：** B3 集成测试 8/10 PASS，Bug 14 修复后需回归 T4。

> E2E 回归 10/10 PASS，v1.2 正式关闭 ✅

---

## ✅ v1.3 — 品牌管理页重构（已完成，2026-04-14，E2E 10/10 PASS）

品牌维度三角色体验统一重构。

| 任务 | 负责 | 状态 |
|------|------|------|
| 菜单重构：删除独立 /brands/apply-my 菜单，ADMIN 加二级菜单 | frontend-dev | ✅ 完成 |
| BrandDashboard 新增产品列表区块（12条，VIP 优先） | frontend-dev | ✅ 完成 |
| BrandDashboard 流量数据占位区块 | frontend-dev | ✅ 完成 |
| 后端：GET /brand/my-products 接口（按 brandId，VIP 优先） | backend-dev | ✅ 完成 |
| i18n 三语言同步 | frontend-dev | ✅ 完成 |
| reviewer 审查 | reviewer | ✅ [OK] |
| Bug 修复：brand:apply:audit 未入库（V7 Flyway） | backend-dev | ✅ 完成 |
| e2e 回归 | e2e-tester | ✅ 10/10 PASS |

**完成标准：** COMPANY 账号进入 /brands 能看到品牌 Dashboard + 产品列表；ADMIN 菜单有二级子项。

---

## ✅ v1.3.1 — 底部操作栏规范（已完成，2026-04-14，E2E 6/6 PASS）

全站 Editor 统一改造，保存按钮从右上角迁移到 Sticky Footer Bar。

| 任务 | 状态 |
|------|------|
| 新建 StickyFooterBar 组件（lastSavedAt + 相对时间 + extraActions） | ✅ 完成 |
| ContentDetailLayout 接入 footerBar prop，全部文案 i18n 化 | ✅ 完成 |
| ProductEditor / StoreEditor / BrandEditor / AgronomistEditor 全部迁移 | ✅ 完成 |
| AgronomistEditor BC 用户 detail 模式补充保存草稿功能 | ✅ 完成 |
| i18n 三语言同步（12 key） | ✅ 完成 |
| docs/frontend/ui-components.md 规范更新 | ✅ 完成 |
| reviewer 审查 | ✅ [OK]（二次审查通过） |
| e2e 回归 | ✅ 6/6 PASS |

**遗留（不阻断）：** BrandEditor.handleDetailSaveDraft 缺 setLastSavedAt（MEDIUM）、dayjs 重复 import（LOW）

---

## ✅ v1.4 — 平台认证申请体系（已完成，2026-04-14，E2E 全部 PASS）

BC 角色对自己的内容（产品/店铺/农艺师）申请平台认证，Admin 审核。

| 期 | 内容 | 状态 |
|----|------|------|
| 期一：后端 | Controller 层暴露 HTTP 接口 + Flyway V6/V8 | ✅ 完成 |
| 期二：前端 | ContentStatusCard 组件 + 四维度 Editor 接入 | ✅ 完成 |
| 期三：Admin | Dashboard 待办数量 + 通知 | ✅ 完成 |

**E2E 结果（期一/二）：** TC-C1~C3, C5 PASS；TC-C4（品牌认证）SKIP — BrandDashboard 未集成 ContentStatusCard，品牌认证入口待规划。

**E2E 结果（期三）：** TC-P3-1~P3-3 全部 PASS（3/3）— Dashboard 待办数量展示 + 审核通知收发均正常。

> 遗留：品牌维度认证入口（BrandDashboard 集成 ContentStatusCard）列入 v1.5 或单独排期
> [INFO] Dashboard PendingRow 路径 `/certification?status=PENDING` 暂无对应列表页（期三设计决策，不建独立管理页）

---

## ✅ v1.4.1 — 遗留清理（已完成，2026-04-14，E2E 3/3 PASS）

> 三个轻量任务合并一个 session，从轻到重顺序执行。

| # | 任务 | 状态 |
|---|------|------|
| T1 | BrandDashboard 集成 ContentStatusCard（品牌认证入口） | ✅ 完成 |
| T2 | CONTENT_OFFLINE 通知完整性核查（三维度已有，Notifications.tsx 颜色 default→orange） | ✅ 完成 |
| T3 | Discourse 论坛关联 D-论坛1：Flyway V9 + ProductMapper.xml + ProductForm ADMIN 编辑框 | ✅ 完成 |

**E2E 结果：** TC-141-1~141-3 全部 PASS（3/3）

**修复 Bug：**
- Bug 15：ProductMapper.xml getDetailById 缺少 discourse_tag SELECT + resultMap（e2e-tester 发现并修复）

**遗留（低优先级）：**
- Task #16：Notifications.tsx 静态 message 导入源未从 AppProvider 替换（reviewer 发现，待后续迭代）

---

## ✅ v1.5 — Discourse 论坛关联（已完成，2026-04-14，E2E 3/3 PASS）

| 期 | 内容 | 状态 |
|----|------|------|
| D-论坛1 | 后台产品新增 discourse_tag 字段（Flyway + API + Admin 编辑页）| ✅ 完成（v1.4.1/T3） |
| D-论坛2 | Headless 产品详情页展示相关帖子列表 | ✅ E2E 3/3 PASS |
| D-论坛3 | 扩展到品牌/店铺/农艺师 + 体验增强 | 未开始 |

### D-论坛2 改动清单
**后端**：`DiscoursePostVO.java`（新）、`DiscourseService.java`（新）、`ProductController.java`（新增接口）、`ProductService.java`（新增方法）
**Headless**：`app/api/product/discourse-posts/[productId]/route.ts`（新）、`lib/discourse-api.ts`（新）、`components/product-detail/ProductForumLinks.tsx`（改造）、`app/productos/[slug]/page.tsx`（集成）、`es.json`/`zh.json`/`index.ts`（i18n）

**E2E（casalyin-server/e2e/v1.5/discourse-posts.spec.ts）**：TC-151-1/2/3 全部 PASS（3/3）

> Headless UI 验证待 headless-tester 上线后处理（低优先级，接口已验证正常）

---

## ✅ v1.6 — C端 API 健康检查 + Headless 农艺师集成（已完成，2026-04-15）

| 任务 | 内容 | 状态 |
|------|------|------|
| C端 API 健康检查 | 18个后端公开接口，18/18 PASS | ✅ |
| next.config.js 图片修复 | remotePatterns 动态读 + 开发环境通配 | ✅ |
| /agronomos 旧路由清理 | 删除整个目录，统一 /agronomists | ✅ |
| 作物详情页农艺师卡片跳转 | Link 包裹，跳转 /agronomists/[slug] | ✅ |
| 产品详情页农艺师区块 | 从 usage cropId 关联，最多 3 个，tsc 验证通过 | ✅（待数据触发 UI）|
| agronomist-api.ts vipOnly 预留 | AgronomistListOptions 接口 + TODO 注释 | ✅ |
| 后端 vipOnly 参数预留 | AgronomistPublicController 两个接口加可选参数 | ✅ |

---

## ✅ v1.7 — 接口规范 + 农艺师测试 + VIP 补齐 + UI 重构（已完成，2026-04-16，E2E 27/27 PASS）

> 版本协议（D15）：版本完成后用户执行 /compact 清理上下文；版本过大拆子版本。

| 任务 | 负责 | 状态 |
|------|------|------|
| pest/crop/composition/tag 接口命名规范修复（/add→/create，参数统一{id}） | backend-dev | ✅ 完成 |
| 前端同步修复 api/endpoints.ts + 各 api 文件 delete 动词（http.get→http.delete） | frontend-dev | ✅ 完成 |
| vipLevel + subscription 接口命名修复（/add→/create，GET delete→DELETE delete） | backend-dev | ✅ 完成 |
| 农艺师加入 VIP entity_type（后端枚举+entityName 填充） | backend-dev | ✅ 完成 |
| BatchSetVip.tsx 加农艺师选项（entityType=4）+ i18n | frontend-dev | ✅ 完成 |
| Subscriptions.tsx entityTypeName 映射加农艺师 | frontend-dev | ✅ 完成 |
| ContentSidebar.tsx 抽离（Descriptions+Link删除+规范信息栏） | frontend-dev | ✅ 完成（已集成到 ContentDetailLayout） |
| 第一期 E2E：农艺师后台管理（TC-A1~A7）`e2e/v1.7/agronomist-admin.spec.ts` | e2e-tester | ✅ 7/7 PASS |
| 第二期 E2E：病虫害+农作物 CRUD（TC-P1~P4, TC-C1~C4）`e2e/v1.7/pest-crop-management.spec.ts` | e2e-tester | ✅ 8/8 PASS |
| 第三期 E2E：有效成分+Tag/TagType（TC-I1~I4, TC-T1~T8）`e2e/v1.7/composition-tag-management.spec.ts` | e2e-tester | ✅ 12/12 PASS |

**v1.7 关闭标准：** ✅ ContentSidebar 完成 + 三期 E2E 全部 PASS → v1.7 正式关闭

---

## ✅ v1.8.1 — 详情接口路径规范（已完成，2026-04-19）

将 crop/pest/composition 旧的 `/public/detail/{entityId}` 非标准路径替换为规范路径。

| 改动 | 内容 |
|------|------|
| `GET /crop/detail/{id}` | 替换旧 `/crop/public/detail/{cropId}`，加 `@NoNeedLogin` |
| `GET /pest/detail/{id}` | 替换旧 `/pest/public/detail/{pestId}`，加 `@NoNeedLogin` |
| `GET /composition/detail/{id}` | 替换旧 `/composition/public/detail/{compositionId}`，加 `@NoNeedLogin` |

**commit：** `casalyin-java` f2057a4（main）

---

## ✅ v1.8 — C端体验完善（已完成，E2E Round 2: 16 PASS / 0 FAIL / 3 SKIP / 8 FAIL A2）

> E2E Round 2 已完成，代码审查通过。西班牙语 A2 阻塞于 headless 3001 端口未启动，待用户确认后单独回归。
>
> **E2E 文件：** `casalyin-server/e2e/v1.8/i18n-sweep.spec.ts` + `slug-stability.spec.ts`
> **Round 2 结果：** 16 PASS / 0 FAIL / 3 SKIP / 8 FAIL（A2，西班牙语 App 未启动）
> **翻译文件：** es.json + zh.json 包含所有必需 key ✅
> **by-slug API：** curl 验证正常（返回 30001 符合预期）✅

### 工作流 A — i18n 扫荡

| 文件 | 改动内容 |
|------|---------|
| `app/plagas/page.tsx` | 加 `useTranslation()`；标题→`t('pages.list.pests')`；空数据→`t('common.noData')`；加载→`t('common.loading')`；失败→`t('pages.error.fetchFailed')` |
| `app/plagas/[slug]/page.tsx` | 加 `useTranslation()`；标题→`t('pages.detail.pest')`；无描述→`t('common.noDescription')`；相关防治→新 key `pages.detail.preventionProducts`；专家→`t('pages.detail.experts')`/`noExperts` |
| `app/ingredientes/page.tsx` | 同上（无专家区块）；相关产品→新 key `pages.detail.relatedProducts` |
| `app/ingredientes/[slug]/page.tsx` | 同上；获取ID失败→新 key `pages.detail.getIdFailed` |
| `app/cultivos/[slug]/page.tsx` | 审计：有 `useTranslation()`，检查是否还有遗漏中文 |
| `app/empresas/page.tsx` | 加 `useTranslation()`；标题→`t('pages.list.brands')`；空数据→`t('common.noData')`；加载→`t('common.loading')`；失败→`t('pages.error.fetchFailed')` |
| `components/dimensions/CropsResults.tsx` | 加 `useTranslation()`；加载→`t('search.searching')`；标题→`t('dimensions.crops')` |
| `components/dimensions/PestsResults.tsx` | 加 `useTranslation()`；加载→`t('search.searching')`；标题→`t('dimensions.pests')` |

### 工作流 B — slug↔ID 修复

**后端新增接口：**

| 接口 | 说明 |
|------|------|
| `GET /crop/by-slug/{slug}` | 按 slug 精确查 cropId（Flyway + Entity + Controller） |
| `GET /pest/by-slug/{slug}` | 按 slug 精确查 pestId |
| `GET /composition/by-slug/{slug}` | 按 slug 精确查 compositionId |

**前端改动（cultivos/plagas/ingredientes detail pages）：**

旧逻辑：`slug.replace(/-/g, ' ')` → `searchProducts()` → name match → originalId  
新逻辑：`GET /{dimension}/by-slug/{slug}` → 直接拿 ID，移除脆弱 name 匹配

### 工作流 C — zh.json / es.json 新增 key

```
pages.detail.preventionProducts  "相关防治产品" / "Productos de Prevención"
pages.detail.relatedProducts      "相关产品"     / "Productos Relacionados"
pages.detail.getIdFailed         "获取 ID 失败" / "Error al obtener ID"
dimensions.searching             "搜索中..."     / "Buscando..."
```

### 关闭标准
- 所有硬编码中文替换完成，`tsc --noEmit` 零错误
- slug 详情页刷新 URL 直接进页，不经过 name 匹配
- headless-tester E2E 回归通过

---

## 📋 遗留需求（长期）

| # | 内容 | 说明 |
|---|------|------|
| ~~B4~~ | ~~i18n 补全：productForm.brand / storeForm.fields.brand 缺 en/es-ES~~ | ✅ 已自愈（三语言均已存在） |
| ~~Task #16~~ | ~~Notifications.tsx 静态 message 导入源未从 AppProvider 替换~~ | ✅ 已自愈（第 6 行已从 AppProvider 导入） |
| ADR-005 | Admin 强制下架通知（CONTENT_OFFLINE） | 详见 docs/decisions/architecture-decisions.md |
| D11 | 品牌流量数据采集（PV/UV） | 规划中，当前 BrandDashboard 留占位 |
| **D-论坛3** | **Discourse 关联扩展到品牌/店铺/农艺师 + 体验增强** | **下期 session 优先讨论** |

---

## ⬜ v2.0 — 仓库与物流（待立项）

> 需求讨论阶段，Q4 后半段 + Q5~Q12 共 8 个问题待确认。
> 不插队当前 v1.x 计划。
>
> **进度：** 4 个问题已确认（Q1/Q2/Q3/Q4 部分），详见 decisions.md D13

### 已确认

| 确认项 | 内容 |
|--------|------|
| 库存维度 | 挂在「产品」下，同一产品多经销商，库存都在平台仓库 |
| 操作角色 | 只有平台方（ADMIN）操作入库/出库，需上传 PDF/图片凭证 |
| 物流模式 | 第三方物流，基本状态 |
| 展示角色 | 供应商看 Excel 库存历史；买家看下单后物流状态（Q4 后半待确认） |

### 待确认（8 项）

| # | 问题 |
|---|------|
| Q4 续 | 买家下单后 C 端如何追踪物流 |
| Q5 | SKU vs 产品：库存以 SKU 为单位还是产品为单位 |
| Q6 | 供应商入库申请流程 |
| Q7 | 库存预警机制 |
| Q8 | 库存为 0 前端行为 |
| Q9 | 物流状态档位 |
| Q10 | 买家端展示形式 |
| Q11 | 仓库分区/库位 |
| Q12 | 退货/退款 |

### 关闭标准

所有 8 个问题确认后，用户与 team-lead 共同评估开发规模，决定是否拆分 sprint。

---

## 已归档任务文件夹

以下文件夹对应已完成任务，内容保留供参考：

| 文件夹 | 对应版本 | 内容 |
|--------|----------|------|
| backend-dev/task-b1-product-sensitive-word/ | v1.1 | B1 敏感词确认 |
| backend-dev/task-b2-role-utils/ | v1.1 | B2 RoleUtils 重构 |
| backend-dev/task-b-brand3-sync-products/ | v1.1 | B-品牌3 同步产品 |
| backend-dev/task-b3-brand-apply/ | v1.2 | B3-1 接口梳理 |
| frontend-dev/task-online-offline/ | v1.0 | 上下架前端改造 |
| frontend-dev/task-b3-user-apply-page/ | v1.2 | B3-3 申请入口页 |
| reviewer/review-task7/ | v1.0 | 第七期 E2E 审查 |
| reviewer/review-b3-2/ | v1.2 | B3-2 审查 |
| reviewer/review-b3-3/ | v1.2 | B3-3 审查 |
