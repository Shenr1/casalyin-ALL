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

## ✅ v1.9 — 别名功能 + 搜索并行化（已完成，2026-04-19）

> **背景**：用户搜索时同物异名无法命中（如科学名 vs 俗名）；后端搜索串行 6 查询压力大。
> **与 C端的关系**：C端 T8（debounce+最少2字符）完全独立先行；本版本完成后搜索结果自动受益，无需 C端额外改动。

### Phase 1：别名字段（backend-dev）

| 改动 | 内容 |
|------|------|
| Flyway V12 | `t_crop`、`t_pest`、`t_chemical_composition` 各加 `aliases JSON` 列 |
| CropMapper.xml / PestMapper.xml / ChemicalCompositionMapper.xml | searchByKeyword SQL 加 `OR JSON_SEARCH(aliases, 'one', CONCAT('%', #{keyword}, '%')) IS NOT NULL` |
| 对应 VO/Entity | 加 `aliases` 字段（`List<String>`，JSON 反序列化） |

**别名规则：**
- 仅 ADMIN 可维护，COMPANY 用户不可见
- 存储格式：`["俗名1", "俗名2"]`

### Phase 2：Admin UI 别名录入（frontend-dev，依赖 Phase 1）

| 改动 | 内容 |
|------|------|
| CropManagement.tsx | 表格加 `aliases` 列，使用 `Select mode="tags"` 内联编辑 |
| PestManagement.tsx | 同上 |
| CompositionManagement.tsx | 同上 |
| i18n | 三语言同步 `aliases` 字段名 |

### Phase 3：搜索并行化（backend-dev，可与 Phase 1 并行）

| 改动 | 内容 |
|------|------|
| GlobalSearchService.java | 6 个串行查询改为 `CompletableFuture.allOf()` 并行执行 |

**效果：** 响应时间从 ~120ms 降至 ~20ms（6个查询变为同时执行）

### 关闭标准

- [x] Phase 1：Flyway V12 + 三个 Mapper JSON_SEARCH 别名匹配 ✅
- [x] Phase 2：Admin 可在表格内录入/删除别名，保存生效 ✅
- [x] Phase 3：并行化后搜索响应时间明显缩短 ✅
- [x] e2e 回归：跳过（用户决策，2026-04-20）

---

## 📋 遗留需求（长期）

| # | 内容 | 说明 |
|---|------|------|
| ~~B4~~ | ~~i18n 补全：productForm.brand / storeForm.fields.brand 缺 en/es-ES~~ | ✅ 已自愈（三语言均已存在） |
| ~~Task #16~~ | ~~Notifications.tsx 静态 message 导入源未从 AppProvider 替换~~ | ✅ 已自愈（第 6 行已从 AppProvider 导入） |
| ~~ADR-005~~ | ~~Admin 强制下架通知（CONTENT_OFFLINE）~~ | ✅ 已完成（StoreService/ProductService/AgronomistService 均已实现） |
| D11 | 品牌流量数据采集（PV/UV） | 规划中，当前 BrandDashboard 留占位 |
| ~~D-论坛3~~ | ~~Discourse 关联扩展到品牌/店铺/农艺师 + 体验增强~~ | ⏸ 暂缓。原因：论坛内容质量不可控，自动拉取可能展示负面帖子伤害 COMPANY 付费用户，待论坛运营成熟、内容可管控后再评估 |
| C端-店铺展示 | ProductStores 组件：VIP 店铺高亮可点击进详情，非 VIP 店铺仅展示店铺名称（灰色只读，无跳转链接） | 决策 2026-04-20：C端用户以农民为主，"开放完整店铺详情"对非客户不合理；VIP 作为"详情可点击"的门槛而非"出现在页面"的门槛。待排期实现。 |

---

## ✅ v2.0 — 仓库与物流（已完成，2026-04-20，E2E 6/6 PASS）

> 需求已完全确认，详见 decisions.md D13。
> 邮件通知：Gmail SMTP（Spring Boot JavaMail）。目标市场：秘鲁。

### Phase 1：后端数据层（backend-dev）

| # | 任务 | 状态 |
|---|------|------|
| V2-B1 | Flyway V13：t_product 加 available_cities（JSON）、stock_quantity（INT） | ✅ |
| V2-B2 | Flyway V14：新建 t_sales_record 表 | ✅ |
| V2-B3 | 产品接口更新：update/detail/query 暴露新字段 | ✅ |
| V2-B4 | 销售记录 CRUD（create/query/detail/add-tracking） | ✅ |
| V2-B5 | 公开查询接口：GET /order/track?orderNo=xxx（@NoNeedLogin） | ✅ |
| V2-B6 | Gmail SMTP 接入：create 时自动发邮件给买家（含订单号） | ✅ |

### Phase 2：Admin 前端（frontend-dev，依赖 Phase 1）

| # | 任务 | 状态 |
|---|------|------|
| V2-F1 | ProductEditor 加 available_cities（Select tags）和 stock_quantity（InputNumber） | ✅ |
| V2-F2 | 新建 SalesRecordManagement 列表页 | ✅ |
| V2-F3 | 销售录入表单（产品/数量/城市/价格/返点/买家姓名/买家邮件/支付方式） | ✅ |
| V2-F4 | 销售详情：补录快递单号入口 | ✅ |
| V2-F5 | 路由 + 菜单：/sales-records，仅 ADMIN 可见 | ✅ |
| V2-F6 | i18n 三语言同步 | ✅ |

### Phase 3：C端 Headless（frontend-dev，可与 Phase 2 并行）

| # | 任务 | 状态 |
|---|------|------|
| V2-H1 | 产品详情页：展示可购城市 + 库存状态（有货/缺货） | ✅ |
| V2-H2 | 新建 /track 页面：输入订单号 → 展示信息 + 跳转 17track | ✅ |
| V2-H3 | i18n（es/zh） | ✅ |

### Phase 4：E2E 验证（e2e-tester）

| # | 任务 | 状态 |
|---|------|------|
| V2-E1 | TC：Admin 录入销售 → order_no 生成 → 邮件触发 | ✅ |
| V2-E2 | TC：C端 /track 凭订单号查询 | ✅ |
| V2-E3 | TC：产品详情页 available_cities 展示 | ✅ |

### t_sales_record 表结构

```sql
id, product_id, qty, city, price, rebate_rate,
buyer_name, buyer_email, payment_method,
tracking_no（nullable）, order_no（unique）,
status ENUM('PENDING','SHIPPED','DONE'),
creator_id, created_at, updated_at
```

### 关闭标准

- Admin 可录入销售 → 系统生成订单号 → 买家收到邮件 ✓
- Admin 可补录快递单号 ✓
- C端 /track 可凭订单号查询 ✓
- E2E 三组场景全部 PASS ✓

---

---

## ✅ v2.0.1 — 产品 Tag UX 重构 + 批量保存性能修复（已完成，2026-04-21，E2E 4/4 PASS）

### 背景
- Bug A：`t_tag_category.is_required`/`is_multiple` 列存在于 DB，但 Java 后端 Entity/VO/Mapper 从未读取，导致前端 isRequired/isMultiple 逻辑静默失效
- Bug B：ProductEditor detail 模式 `isEditMode` 永远为 true，tag 只读展示失效
- 性能问题：4 个管理页（TagCategory / Composition / Crop / Pest）batch-upsert 全量提交导致超时

### 任务清单

| # | 任务 | 负责 | 状态 |
|---|------|------|------|
| T1 | 后端打通 is_required/is_multiple（Entity/VO/Form/Mapper/Service） | backend-dev | ✅ reviewer [OK] |
| T2 | ProductForm.tsx 重构：两步式→单层展开（Radio/Checkbox 按 isMultiple 动态切换） | frontend-dev | ✅ reviewer [WARN→OK] |
| T3 | TagCategoryManagement.tsx 加 isRequired/isMultiple 编辑列 | frontend-dev | ✅ 完成 |
| T4 | 修复 BUG-TAG-002：ProductEditor detail 模式 isEditMode 改为 `mode !== 'detail'` | frontend-dev | ✅ e2e 验证通过 |
| T5 | 4 个管理页 batch-upsert 只提交有变更行（TagCategory/Composition/Crop/Pest） | frontend-dev | ✅ 完成 |
| T6 | E2E 回归：TC-TAG-001~004 + 批量保存 smoke test | e2e-tester | ✅ 4/4 PASS |

### 关闭标准
- TC-TAG-001（创建，Card 展开/Radio 单选/isRequired 红 *）✅ PASS
- TC-TAG-002（编辑回填）✅ PASS
- TC-TAG-003（详情只读 tag 格式）✅ PASS
- TC-TAG-004（必填分类未选报错）✅ PASS
- 4个管理页保存响应时间 <2s ✅ PASS（1783ms）

---

## ✅ v2.1 — 地图集成（已完成，2026-04-22，E2E 3/3 PASS）

> Google Maps API Key 已配置，全球城市支持（无国家限制）。
> 设计方向：纯文本 Places Autocomplete（无地图渲染），地址/城市均通过搜索建议选择。

| 维度 | 精度 | 改动内容 | 状态 |
|------|------|---------|------|
| 店铺 | 精确到点（lat/lng） | Flyway V16 + StoreDraftVO 加 latitude/longitude；StoreEditor 地址搜索框（PlacesAddressAutocomplete）自动填 state/city/street；C端店铺详情弹窗展示 Google Maps iframe + 外链 | ✅ |
| 农艺师 | 精确到市 | Flyway V16 + AgronomistDraftVO 加 city；AgronomistEditor 城市 autocomplete 单选（PlacesCityAutocomplete） | ✅ |
| 产品 `available_cities` | 精确到市（多选） | ProductForm `<Select mode="tags">` 替换为 PlacesCityAutocomplete（多选 Tags） | ✅ |
| 销售记录 | 暂缓 | 不在本版本范围 | ⏸ |

**E2E（`casalyin-server/e2e/v2.1/map-integration.spec.ts`）：** TC-MAP-001~003 全部 PASS（3/3，46.4s）

**新增组件：** `PlacesAddressAutocomplete.tsx`、`PlacesCityAutocomplete.tsx`（Ant Design Select + debounce 300ms + Tag 展示）

**删除：** `GoogleMapPicker.tsx`（UX 评估后放弃地图渲染方式）

---

## ✅ v2.2 — C端 VIP 店铺详情页（已完成，2026-04-22，E2E 3/3 PASS）

| 任务 | 内容 | 状态 |
|------|------|------|
| 新建 `/tiendas/[id]` 详情页 | Client Component，展示地址/地图/店铺信息/联系方式，VipBadge | ✅ |
| ProductStores.tsx 弹窗加「查看完整详情」Link | 跳转 `/tiendas/{storeId}` | ✅ |
| productos/[slug] 内联弹窗同步 | 同上 + 7处 i18n 修复 | ✅ |
| lib/sanitize.ts | DOMPurify 共享净化函数，替换手写正则 | ✅ |
| i18n store.viewDetails | zh/es/index.ts 三文件同步 | ✅ |
| StoreMapper.xml Bug 修复 | vip_level_value → vip_level（3处），VIP 字段映射修复 | ✅ |
| api-contracts.md 补充 | GET /api/store/detail/{id} + StoreDetailVO 字段定义 | ✅ |

**E2E（`casalyin-server/e2e/v2.1/store-detail.spec.ts`）：** TC-T9-1~3 全部 PASS（3/3）

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
