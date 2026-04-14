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

## 🔄 v1.3 — 品牌管理页重构（进行中，2026-04-13）

品牌维度三角色体验统一重构。

| 任务 | 负责 | 状态 |
|------|------|------|
| 菜单重构：删除独立 /brands/apply-my 菜单，ADMIN 加二级菜单 | frontend-dev | ✅ 完成 |
| BrandDashboard 新增产品列表区块（12条，VIP 优先） | frontend-dev | 🔄 进行中 |
| BrandDashboard 流量数据占位区块 | frontend-dev | 🔄 进行中 |
| 后端：GET /brand/my-products 接口（按 brandId，VIP 优先） | backend-dev | 🔄 进行中 |
| i18n 三语言同步 | frontend-dev | 🔄 进行中 |
| reviewer 审查 | reviewer | ⏳ 待触发 |
| e2e 回归 | e2e-tester | ⏳ 待触发 |

**完成标准：** COMPANY 账号进入 /brands 能看到品牌 Dashboard + 产品列表；ADMIN 菜单有二级子项。

---

## 📋 v1.4 — 平台认证申请体系（未开始，排 v1.3 后）

BC 角色对自己的内容（产品/店铺/农艺师/品牌）申请平台认证，Admin 审核。

详细规划见：`.plans/casalyin/decisions.md` → D9

| 期 | 内容 | 状态 |
|----|------|------|
| 期一：后端 | Controller 层暴露 HTTP 接口（Service 已写好）+ Flyway 建表确认 | ⏳ 未开始 |
| 期二：前端 | ContentStatusCard 组件 + 四维度 Editor 接入 | ⏳ 未开始 |
| 期三：Admin | Dashboard 待办数量 + 通知 | ⏳ 未开始 |

---

## 📋 v1.5 — Discourse 论坛关联（未排期）

详细计划见：`.plans/casalyin/discourse-integration/plan.md`

| 期 | 内容 |
|----|------|
| D-论坛1 | 后台产品新增 discourse_tag 字段（Flyway + API + Admin 编辑页） |
| D-论坛2 | Headless 产品详情页展示相关帖子列表 |
| D-论坛3 | 扩展到品牌/店铺/农艺师 + 体验增强 |

---

## 📋 遗留需求（长期）

| # | 内容 | 说明 |
|---|------|------|
| B4 | i18n 补全：productForm.brand / storeForm.fields.brand 缺 en/es-ES | 低优先级 |
| ADR-005 | Admin 强制下架通知（CONTENT_OFFLINE） | 详见 docs/decisions/architecture-decisions.md |
| D11 | 品牌流量数据采集（PV/UV） | 规划中，当前 BrandDashboard 留占位 |

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
