# Discourse 论坛关联功能 — 产品规划

> 状态: PLANNING（未开发）
> 创建: 2026-04-13
> 优先级: 待定
> 涉及项目: casalyin-java（后台后端）、casalyin-server（后台前端）、casalyin-Headless（C端前台）
> Discourse 域名: https://community.casalyin.com/
> Headless 技术栈: Next.js 14 + GraphQL（graphql-request）

---

## 背景与目标

将 Discourse 论坛与产品详情页关联，形成流量闭环：
- 产品详情页 → 论坛：用户看到相关帖子，点击进入 Discourse 参与讨论
- 论坛 → 产品页：论坛帖子导流回产品页（由运营在帖子中附产品链接）

**论坛情况：**
- Discourse 对外公开（访客可读，无需登录）
- 前期以官方/运营发帖为主，中后期有用户自发讨论
- Headless 技术栈：React

---

## 核心机制：Tag 驱动 + 关键词兜底

```
产品页加载 → 取 discourse_tag →
  有 tag → 调 Discourse API /tag/{tag}/l/latest.json
    有结果 → 展示（最多10条）
    无结果 → fallback：按产品名关键词搜索 /search.json?q={name}
              有结果 → 展示（最多10条）
              无结果 → 隐藏「相关讨论」区块（不展示空状态）
```

**discourse_tag 字段**：每个产品在后台有一个 `discourse_tag` 字段，默认由产品 slug 自动生成（如 `product-herbicide-001`），Admin 可手动修改。运营发帖时给帖子打上对应 tag，产品页自动展示。

---

## 展示设计（Headless 产品详情页）

```
─────────────────────────────────────────
📋 相关讨论                   [前往论坛 →]
─────────────────────────────────────────
• 帖子标题 A            💬 12  👁 340  3天前
• 帖子标题 B            💬 5   👁 128  1周前
• 帖子标题 C            💬 0   👁 56   2周前
  ... （最多展示10条）
─────────────────────────────────────────
```

- 标题点击：新标签页打开 Discourse 对应帖子
- 「前往论坛」：跳转到该 tag 的论坛频道页，引导用户参与讨论
- 无相关帖子时：整个区块不渲染

---

## 分期计划

### 第一期：后台字段支持（casalyin-java + casalyin-server）

**目标**：让每个产品可以配置 `discourse_tag`，API 返回该字段。

**后端（casalyin-java）**：
- Flyway 迁移：`t_product` 表新增 `discourse_tag` VARCHAR(100) NULL
- ProductDetailVO 新增 `discourseTag` 字段，detail 接口返回
- ProductService：saveDraft / approve 时自动生成默认 tag（基于产品名 slugify，或 `product-{id}`）

**前端（casalyin-server）**：
- ProductEditor 编辑页新增「Discourse Tag」输入框
  - 仅 ADMIN 可见和编辑（BASE/COMPANY 不展示）
  - placeholder：自动生成的默认值
  - 说明文案：「Discourse 论坛对应 Tag，用于关联相关帖子」

**验收标准**：
- [ ] ADMIN 可在产品编辑页查看并修改 discourse_tag
- [ ] 产品 detail API 返回 discourseTag 字段
- [ ] 数据库有对应字段和 Flyway 迁移文件

---

### 第二期：Headless 前台展示

**目标**：产品详情页底部展示最多10条相关 Discourse 帖子。

**技术方案**：
- 服务端调用 Discourse API（SSR / getServerSideProps 或 Server Component）
- **不从客户端直接调**，避免 CORS 和 Discourse 限流（匿名 API ~10 req/min）
- 缓存策略：Next.js `revalidate: 1800`（30分钟）或 Redis 缓存
- API 调用链：
  1. 取 discourseTag → `GET {DISCOURSE_URL}/tag/{tag}/l/latest.json`
  2. 无结果 → `GET {DISCOURSE_URL}/search.json?q={productName}`
  3. 取前10条，格式化字段：title / slug / reply_count / views / created_at / url

**Headless 改动**：
- 产品详情页底部新增 `<DiscussionList />` 组件
- 组件接收 `posts` 数组（服务端已取好，空数组则不渲染）
- 样式：简洁列表，每行：标题 + 回复数 + 浏览数 + 时间

**环境变量**：
- `DISCOURSE_URL=https://community.casalyin.com`（服务端用，不暴露给客户端）

**Next.js 14 缓存方案**（Server Component）：
```ts
// 在 Server Component 中，带 ISR 缓存
const posts = await fetch(
  `https://community.casalyin.com/tag/${tag}/l/latest.json`,
  { next: { revalidate: 1800 } }  // 30分钟缓存
).then(r => r.json())
```
无需 Redis，Next.js 原生 fetch 缓存直接支持，零额外依赖。

**⚠️ GraphQL schema 注意**：
Headless 通过 GraphQL 从 casalyin-java 获取产品数据。第一期在 `t_product` 加完 `discourse_tag` 字段后，casalyin-java 的 GraphQL schema（ProductType）也必须同步暴露 `discourseTag` 字段，否则 Next.js Server Component 无法拿到该值。

**验收标准**：
- [ ] 产品详情页底部正确展示相关帖子列表（有数据时）
- [ ] 无相关帖子时区块不渲染
- [ ] 点击帖子标题新标签页打开 Discourse
- [ ] 「前往论坛」按钮跳转到对应 tag 页面
- [ ] 30分钟缓存生效（不每次请求 Discourse）

---

### 第三期：扩展维度 + 体验增强（低优先级）

**扩展到其他维度**：
- `t_brand`、`t_store`、`t_agronomist` 也加 `discourse_tag` 字段
- 对应编辑页加配置入口
- Headless 品牌/店铺/农艺师详情页也展示相关讨论

**体验增强**：
- 「参与讨论」CTA 按钮（跳转到 Discourse 新建帖子页，带 tag 预填）
- 展示帖子的第一条回复预览（需调 `/t/{id}.json` 获取内容）
- Admin 批量设置已有产品的 discourse_tag（避免逐条编辑）

**可选（视论坛活跃度决定）**：
- Discourse Embed：在产品页嵌入 Discourse 评论区（官方功能），用户可直接在产品页发帖，内容同步到 Discourse

---

## 依赖和前置条件

| 条件 | 说明 |
|------|------|
| Discourse 已部署 | ✅ 域名：https://community.casalyin.com/ |
| Discourse 对外公开 | ✅ 已确认，匿名 API 可读 |
| Headless 技术栈 | ✅ Next.js 14 + GraphQL（graphql-request） |
| Headless 通过 GraphQL 拿产品数据 | ⚠️ 第一期需要 casalyin-java 的 GraphQL schema 同步加 discourseTag 字段，否则 Headless 拿不到 |
| 产品有 slug 机制 | 若无，discourse_tag 默认用 `product-{id}` |

---

## 技术风险

| 风险 | 应对 |
|------|------|
| Discourse API 限流（匿名10 req/min） | 服务端缓存30分钟，不直接在客户端调 |
| 产品名模糊导致 fallback 误匹配 | fallback 仅作兜底，优先依赖 tag 精确匹配；误匹配时 Admin 可手动清空 discourse_tag |
| Discourse 宕机影响产品页 | API 调用加 try/catch，失败时静默隐藏区块，不影响产品页主体加载 |
| Headless 是 SPA 非 SSR | 若无服务端，改为 API Route 中转（Next.js）或 casalyin-java 做代理接口 |

---

## 待确认事项（开发前需决策）

- [ ] Headless 是 Next.js 还是 Vite SPA？（影响缓存方案）
- [ ] Discourse 论坛域名是什么？
- [ ] 第一期预计什么时候开始？（当前 B3 测试完成后？还是等 Headless 进入开发节奏？）
- [ ] discourse_tag 默认值生成规则：用产品名 slugify，还是用 `product-{id}` 更保险？

---

## 工作量估算

| 期次 | 后端 | 前台后端（casalyin-server） | Headless | 总计 |
|------|------|--------------------------|----------|------|
| 第一期 | 小（1 Flyway + 1 VO 字段） | 小（1 个输入框） | 无 | **0.5天** |
| 第二期 | 无 | 无 | 中（组件+API调用+缓存） | **1~1.5天** |
| 第三期 | 小（3张表各加字段） | 小（3个编辑页各加输入框） | 中（3个详情页加组件） | **1~2天** |
