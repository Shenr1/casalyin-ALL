# 交接文档 — 2026-04-14（v1.5 D-论坛2 代码完成）

> 用途：/compact 或 session 重启后恢复状态用

---

## 当前版本状态

| 版本 | 状态 |
|------|------|
| v1.0 ~ v1.4.1 | ✅ 全部完成 |
| 规范清理：全站 19 个文件静态 message 导入 | ✅ 完成，tsc 零错误，reviewer [OK] |
| v1.5 D-论坛2：代码实现 | ✅ 后端 + Headless 前端均完成 |
| v1.5 D-论坛2：E2E 验证 | ⏳ 待完成（后端 Rebuild 后验证） |

---

## ⚠️ 下次 session 第一件事

**后端需要 Rebuild Project**（IntelliJ → Build → Rebuild Project）再启动，否则有编译缓存问题（STORE_LOCKED 找不到）。代码本身没有 bug，纯缓存问题。

重启后 e2e-tester 跑：
- TC-151-1：有 discourse_tag 产品 → `GET /product/discourse-posts/{id}` 返回帖子列表
- TC-151-2：无 discourse_tag 产品 → 返回 []
- TC-151-3：不存在产品 ID → 返回 []

---

## v1.5 D-论坛2 改动文件清单

### 后端（casalyin-java）
| 文件 | 说明 |
|------|------|
| `product/domain/vo/DiscoursePostVO.java`（新） | 帖子 VO（id/title/slug/createdAt/replyCount/views） |
| `product/service/DiscourseService.java`（新） | 调 `community.casalyin.com` Discourse API，异常返回空列表 |
| `product/controller/ProductController.java` | 新增 `@NoNeedLogin GET /product/discourse-posts/{id}` |
| `product/service/ProductService.java` | 新增 `getDiscoursePosts(Long productId)` |

### Headless（casalyin-Headless）
| 文件 | 说明 |
|------|------|
| `app/api/product/discourse-posts/[productId]/route.ts`（新） | Route Handler，代理到后端 |
| `lib/discourse-api.ts`（新） | `getProductDiscussions(productId)`，失败返回 [] |
| `components/product-detail/ProductForumLinks.tsx` | 改为真实数据，新增 loading/空状态/回复数/浏览数 |
| `app/productos/[slug]/page.tsx` | 传 `productId` 给 ProductForumLinks |
| `lib/i18n/translations/es.json` | 新增 `product.forum.*` 6 个 key |
| `lib/i18n/translations/zh.json` | 新增 `product.forum.*` 6 个 key |
| `lib/i18n/index.ts` | 新增 TranslationKey 类型声明 |

---

## 下一步候选方向

| 优先级 | 内容 |
|--------|------|
| 最高 | 完成 v1.5 E2E：Rebuild 后端 → e2e-tester 验证 TC-151-1/2/3 |
| 高 | 招募 headless-dev + headless-tester，做 C 端 API 健康检查 |
| 待定 | v1.5 D-论坛3：扩展到品牌/店铺/农艺师 + 体验增强 |
| 待定 | v2.0 仓储物流：先对齐 D13 的 5 个问题再立项 |

---

## 团队规划（已决策）

现有团队 `casalyin-20260414` 新增两个专职角色：

| 角色 | 职责 |
|------|------|
| `headless-dev` | 专职 Next.js Headless 开发（不碰 casalyin-server） |
| `headless-tester` | 专职 C 端 Playwright 测试，首要任务：C 端 API 健康检查 |

backend-dev / reviewer 两个角色继续共享。

---

## 重启团队后第一步

1. 读 `current-team.txt` 拿团队名（casalyin-20260414）
2. SendMessage 尝试唤醒已有 agent
3. 读本文件 + task_plan.md 确认当前进度
4. Rebuild 后端 → e2e-tester 验证 v1.5

---

## 技术栈快速参考

- 后端端口：8690（dev profile）
- Flyway 最新版本：V9（discourse_tag）
- Discourse 论坛：https://community.casalyin.com
- Headless：Next.js 14 App Router + Tailwind CSS + TypeScript
- Headless API 模式：lib/*-api.ts → app/api/**/route.ts → 后端
- 通知类型：CONTENT_NOTICE / AUDIT_APPROVED / AUDIT_REJECTED / CONTENT_OFFLINE
- 认证资源类型：PRODUCT / STORE / AGRONOMIST / BRAND
