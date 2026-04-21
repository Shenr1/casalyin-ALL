# e2e-tester - 任务计划

> 角色: 联调测试（Playwright）
> 状态: in_progress
> 当前任务: Task #2 — 【v1.3 E2E】品牌 Dashboard + 产品列表联调回归

## 项目信息

测试地址：http://localhost:5173
测试框架：Playwright（casalyin-server/e2e/）
测试账号：见 docs/testing/test-strategy.md

## 核心协议

- 每次 dev 代码变更后必须运行相关测试
- Bug 记录到 findings.md，通知 team-lead 和相关 dev

---

## Task #2 — 【v1.3 E2E】品牌 Dashboard + 产品列表联调回归

**分配自：** team-lead（2026-04-14）
**状态：** 🟡 等待环境就绪（前后端服务未运行）

### 测试范围

| 组 | 用例 | 覆盖接口/页面 |
|----|------|--------------|
| P1 | TC-P1-001 | POST /brand/apply/submit（Bug 13 最终验证） |
| P1 | TC-P1-002 | POST /product/sync-creator（Bug 14 最终验证） |
| P2 | TC-P2-001~006 | COMPANY /brands → BrandDashboard（信息/订阅/统计/产品列表/行点击/查看全部/流量占位） |
| P3 | TC-P3-001~002 | ADMIN 品牌二级子菜单 / BASE 单入口权限验证 |

### 测试脚本

`casalyin-server/e2e/v1.3/v1-3-acceptance.spec.ts`（10 个用例）

### 依赖检查

- [x] Task #1 前端代码已完成（BrandDashboard 产品列表 + 流量占位 + 路由重构 + i18n）
- [x] 后端接口 GET /brand/my-products 已完成（backend-dev 确认）
- [ ] **前端服务 http://localhost:5173 启动** ← 当前阻塞点
- [ ] **后端 Spring Boot 服务启动**

### 阻塞记录

2026-04-14：测试脚本就绪，运行时 `net::ERR_CONNECTION_REFUSED`，前端 Vite 服务未运行。
已通知 team-lead 请求用户启动服务。

### 结果（待填写）

TBD — 等待服务就绪后运行
