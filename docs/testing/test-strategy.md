# 测试规范

> 最后更新：2026-04-11（登录方式已更新为邮件验证码流程）

---

## 技术栈

- **框架：** Playwright（TypeScript）
- **测试环境：** 本地 `http://localhost:5173`（需先启动前端开发服务）
- **数据库：** `casalyin_admin_v1`（本地 MySQL）

---

## 测试账号

| 角色 | 登录入口 | 邮箱 | 验证码 | 说明 |
|------|----------|------|--------|------|
| B（BASE） | `/login` | `123456@qq.com` | `123456`（本地固定值）| 邮箱 + 邮件验证码，本地后端非生产环境固定通过 |
| C（COMPANY） | `/login` | `123123@qq.com` | `123456`（本地固定值）| 同上；C 角色由 B 角色通过品牌申请升级获得 |
| A（ADMIN） | `/internal/auth-admin-portal` | — | 无验证码 | 账号 `custom` + 密码 `123123`，隐藏入口 |

**登录方式说明：**
- B/C 角色走「邮箱 + 邮件验证码」流程：填邮箱 → 点「Send Code」→ 填验证码（本地固定 `123456`）→ 点「Sign In」
- 本地后端 `EmailVerifyService` 在非生产环境下，输入 `123456` 直接通过，无需真实邮件
- **无注册页**：第一次用邮箱登录即自动注册，注册即登录
- A 角色走账号+密码流程，无验证码字段

---

## 文件结构约定

```
casalyin-server/
  e2e/
    auth/
      login.spec.ts         ← 登录 / Token / 权限隔离（第一期）
    draft/
      product-draft.spec.ts
      store-draft.spec.ts
      ...
    helpers/
      auth.ts               ← loginBase / loginCompany / loginAdmin 封装
      navigation.ts
    online-offline.spec.ts  ← 上下架专项（第五期）
```

---

## 登录封装规范

`e2e/helpers/auth.ts` 提供三个登录函数：

```typescript
// B 角色（BASE）登录 — 邮箱 + 验证码
export async function loginBase(page: Page)

// C 角色（COMPANY）登录 — 邮箱 + 验证码
export async function loginCompany(page: Page)

// A 角色（ADMIN）登录 — 账号 + 密码，隐藏入口
export async function loginAdmin(page: Page)
```

**流程（B/C）：** `page.goto('/login')` → `getByLabel('Email').fill(email)` → 点「Send Code」→ 等按钮变倒计时 → `getByLabel('Verification Code').fill('123456')` → 点「Sign In」

**流程（A）：** `page.goto('/internal/auth-admin-portal')` → `getByLabel('Account').fill('custom')` → `getByLabel('Password').fill('123123')` → 点「Admin Sign In」

---

## e2e-tester 工作规范

1. **每次代码变更后必须运行相关测试脚本**，确认功能正常
2. **新功能 / Bug 修复必须配套测试脚本**，先写测试用例再运行验证
3. 测试脚本放在 `casalyin-server/e2e/` 目录下
4. 测试失败必须记录到 `.plans/` 对应的 findings 文件，并通知 team-lead
5. 测试脚本命名：`{模块}-{功能}.spec.ts`（如 `product-draft.spec.ts`），按功能放入子目录

---

## 断言规范

- **UI 状态断言**：优先用 Ant Design Tag 文字内容（`toContainText`）判断状态
- **跳转验证**：用 `waitForURL` 而非 `waitForTimeout`
- **数据库验证**：通过前端 API 响应或 UI 展示间接验证，不直接连库（Playwright 不直连 MySQL）
- **权限验证**：尝试访问禁止路由时，断言重定向到 `/login` 或出现权限提示

---

## 测试优先级

| 优先级 | 模块 | 原因 |
|--------|------|------|
| P0 | 草稿状态机（四个维度） | 核心业务流程，状态机复杂 |
| P0 | 权限隔离（BASE/COMPANY/ADMIN） | 安全边界 |
| P1 | 上下架操作（四个维度） | 已知有 Bug |
| P1 | 登录 / Token 机制 | 基础鉴权 |
| P2 | VIP 展示逻辑 | 变现核心 |
| P2 | UI 规范回归 | 一致性 |
| P3 | 系统管理（审计日志等） | ADMIN 专项 |

---

## 前置数据要求

运行测试前，数据库需准备：

- 品牌「测试品牌X」已存在主表，且已绑定 COMPANY-C（`123123@qq.com`）
- VIP 等级「VIP1」已由 ADMIN 创建
- BASE-A（`123456@qq.com`）已注册，角色为 BASE
- COMPANY（`123123@qq.com`）已完成 B→C 升级，已认领品牌「测试品牌X」

---

## 已知测试难点

| 问题 | 处理方式 |
|------|----------|
| Ant Design Modal 动画 | `await modal.waitFor({ state: 'visible' })` |
| 富文本编辑器（如 Quill）的输入 | `page.evaluate` 直接操作 DOM |
| 文件上传（营业执照等） | `page.setInputFiles` |
| 跨角色切换（BASE → ADMIN） | 每个角色新开 `browserContext`，避免 cookie 污染 |
