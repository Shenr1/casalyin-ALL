# casalyin-headless - 主计划

> 状态: IN_PROGRESS
> 创建: 2026-04-19
> 更新: 2026-04-19
> 团队: casalyin-headless (frontend-dev, e2e-tester, reviewer)
> 决策记录: .plans/casalyin-headless/decisions.md

---

## 1. 项目概述

casalyin-Headless/ — C 端前台，基于 Next.js 14 App Router + TypeScript + Tailwind CSS。提供产品、品牌、农艺师、作物、病虫害、化学成分等多维度搜索与展示页面，支持响应式（移动端 + 桌面端）。

后端 API 由 casalyin-java 提供（本地端口 8690）。业务规范与后管共用 `docs/business/`。

---

## 2. 文档索引

| 文档 | 位置 | 内容 |
|------|------|------|
| 导航地图 | .plans/casalyin-headless/docs/index.md | sections 和行号（custodian 维护） |
| 架构 | .plans/casalyin-headless/docs/architecture.md | 系统组件、数据流、目录结构 |
| API 契约 | .plans/casalyin-headless/docs/api-contracts.md | 前台调用的后端接口定义 |
| 不变量 | .plans/casalyin-headless/docs/invariants.md | 不可违反的系统边界 |
| 后端接口规范 | docs/backend/api-conventions.md | HTTP 方法、路径命名（共享） |
| 业务规范 | docs/business/ | 权限、状态机、VIP（共享） |

---

## 3. 阶段概览

### 切片原则

将任务分解为**垂直切片**，每个切片覆盖完整的页面功能（组件 + 数据获取 + 样式 + 测试）。

### 阶段

- 阶段 0: 项目探索 — frontend-dev 梳理现有代码库结构，记录现状
- 阶段 1: 核心功能开发 — 按页面模块垂直切片实现
- 阶段 2: 联调测试 — E2E 测试关键用户流程
- 阶段 3: 审查与收尾 — 代码审查裁决 + 规范对齐

---

## 4. 任务汇总

| # | 任务 | 负责人 | 状态 | 计划文件 |
|---|------|--------|------|----------|
| T0 | 探索代码库，梳理现有页面结构和 API 调用 | frontend-dev | ✅ 完成 | .plans/casalyin-headless/frontend-dev/task_plan.md |
| T1 | Phase 1 — 功能 Bug 修复（by-slug 路由、农艺师卡片可点击） | frontend-dev | ✅ 完成 | frontend-dev/findings.md |
| T2 | Phase 2 — 信任状 / 内容完善（BlogLinks 接真实数据、logoUrl、featuredImage） | frontend-dev | ✅ 完成 | frontend-dev/findings.md |
| T3 | Phase 3 — i18n 硬编码清理（empresas/productos/cultivos/agronomists 共 22 处） | frontend-dev | ✅ 完成 | frontend-dev/findings.md |
| T4 | Phase 4 — 代码卫生（删除 crop-pest-api.ts 死代码 + agronomists 冗余路由） | frontend-dev | ✅ 完成 | frontend-dev/findings.md |
| T5 | 后端新增详情接口：GET /crop/detail/{id}、/pest/detail/{id}、/composition/detail/{id} | 后台团队 | ✅ 完成（v1.8.1） | — |
| T6 | 各维度详情页动态拉取 Ghost 博客推荐 | frontend-dev | ✅ 完成（v1.2） | 见下方 T6 设计决策 |
| T7 | 活动日历页 /eventos（需后端新增 API） | — | ⬜ 规划中 | — |
| **T8** | **搜索体验优化：最少 2 字符触发 + 300ms debounce** | frontend-dev | ✅ 完成 | 见下方 v1.1 |

---

## ✅ v1.1 — 搜索体验优化（已完成，2026-04-19，E2E 5/5 PASS）

> **背景**：当前搜索每次击键触发一次请求，后端每次串行发 6 个 SQL 查询，压力大。
> **C 端职责**：控制请求频率（前端层面）；后端并行化由 casalyin 团队独立处理，两侧互不依赖。

### 改动内容

| # | 文件 | 改动 |
|---|------|------|
| 1 | `lib/hooks/useSearch.ts` | `MIN_SEARCH_LENGTH=2` 常量；`performSearch` 短路 <2 字符；暴露 `showMinCharsHint` |
| 2 | `components/SearchSection.tsx` | 300ms debounce（原 1000ms）；showMinCharsHint 时展示提示文案 |
| 3 | `components/CompactSearchBar.tsx` | 300ms debounce（原 1000ms）；展开状态同步展示提示文案 |
| 4 | `lib/i18n/index.ts` | `TranslationKey` 新增 `search.minCharsHint` |
| 5 | `lib/i18n/translations/es.json` | `"minCharsHint": "Escribe al menos 2 letras para buscar"` |
| 6 | `lib/i18n/translations/zh.json` | `"minCharsHint": "请输入至少 2 个字符"` |

### E2E 结果（casalyin-server/e2e/headless/search-debounce.spec.ts）

| TC | 描述 | 结果 |
|----|------|------|
| TC-1 | 输入 1 字符：无请求，提示文案可见 | ✅ PASS |
| TC-2 | 输入 2+ 字符：300ms 后触发请求 | ✅ PASS |
| TC-3 | 快速连续输入 3 字符：只发 1 次请求 | ✅ PASS |
| TC-4 | 清空输入后提示文案消失 | ✅ PASS |
| TC-5 | 从 1 字符追加到 2 字符后才触发请求 | ✅ PASS |

**5/5 PASS，v1.1 正式关闭 ✅**

---

## T6 设计决策（2026-04-20 确认）

**博客平台：** Ghost（`blog.casalyin.com`），独立于 Discourse 论坛。

**推荐逻辑：两层叠加，固定展示 3 篇**

1. 第一层（精准）：按当前页面实体的 slug 去 Ghost Content API 拉取匹配文章
2. 第二层（通用补充）：不足 3 篇时，用打了 `general` tag 的技术类文章补齐
3. 去重后取前 3 篇，附「查看更多博客」链接指向 `blog.casalyin.com`

**覆盖维度：** 农作物、病虫害、化学成分、产品、品牌、农艺师 详情页

**tag 命名规范（运营约定）：**
- 专题文章：打对应实体 slug（复用 `discourse_tag` 同名规范）
- 通用技术文章：打 `general` tag

**接口方案：** C 端直接调 Ghost Content API（无需后端新建接口）
- `GET https://blog.casalyin.com/ghost/api/content/posts/?key=xxx&filter=tag:slug&limit=3`
- 建议加 BFF 代理层隐藏 API Key

**复用现有组件：** 升级 `ProductBlogLinks.tsx` 为通用 `BlogRecommendations.tsx`，各维度详情页复用

---

## 5. 当前阶段

v1.1 已完成（2026-04-19）。C端 T8 搜索优化 E2E 5/5 PASS，正式关闭。后台团队独立推进 v1.9（别名+并行化），完成后 C端搜索自动受益，无需额外改动。

v1.2 已完成（2026-04-20）。T6 Ghost 博客推荐集成，reviewer [OK]，tsc 零错误，正式关闭。

**用户待配置：** 在 `.env.local` 中设置 `GHOST_API_KEY=<Ghost Content API Key>`（Ghost Admin → Settings → Integrations → Content API key）。
