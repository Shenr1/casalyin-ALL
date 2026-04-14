# Casalyin 项目规范

> 本文件会在每次 Claude Code 会话启动时自动加载。
> 规范详情在 `docs/` 目录下，此处只列索引和关键约定。

---

## 项目结构

```
casalyin/
  casalyin-server/     ← 后台前端（React + Vite + TypeScript + Ant Design）
  casalyin-java/       ← 后端（Spring Boot + Sa-Token + MyBatis Plus + Flyway）
  casalyin-Headless/   ← C 端前台（不在本项目维护范围）
  docs/                ← 项目规范文档（团队必读）
```

---

## 规范索引（docs/）

### 业务规范

| 文件 | 内容 |
|------|------|
| `docs/business/roles-permissions.md` | BASE/COMPANY/ADMIN 权限边界、COMPANY 身份流转、订阅与冻结逻辑 |
| `docs/business/workflow-draft.md` | 草稿状态机（DRAFT→PENDING→发布/拒绝）、免审直发、敏感词检测 |
| `docs/business/online-offline.md` | 上下架语义（disabledFlag）、四维度 API 规范、前端交互规范 |
| `docs/business/vip-monetization.md` | VIP 体系、各维度 VIP 效果、VipBanner 规范、商业模式 |

### 前端规范

| 文件 | 内容 |
|------|------|
| `docs/frontend/ui-components.md` | 页面宽度、返回按钮、ContentDetailLayout、Table、Modal/message 使用 |
| `docs/frontend/i18n.md` | 禁止硬编码文案、t() 规范、key 命名、三语言同步 |
| `docs/frontend/routing.md` | 完整路由清单，新增页面必须先查此表 |
| `docs/frontend/editor-architecture.md` | XxxEditor 三合一架构（mode: create/draft/detail） |

### 后端规范

| 文件 | 内容 |
|------|------|
| `docs/backend/api-conventions.md` | HTTP 方法、路径命名、鉴权注解、权限码命名、四维度接口矩阵 |
| `docs/backend/database.md` | Flyway 迁移规范、字段命名约定、核心表结构 |

### 测试规范

| 文件 | 内容 |
|------|------|
| `docs/testing/test-strategy.md` | Playwright 规范、测试账号、文件结构、断言规范 |
| `docs/testing/test-cases.md` | 完整测试用例清单（十一期） |

### 架构决策

| 文件 | 内容 |
|------|------|
| `docs/decisions/architecture-decisions.md` | ADR 记录：Dashboard 精简、XxxEditor、Flyway、上下架统一 |

---

## 核心约定（高频参考，必须遵守）

### 后端接口命名

```
POST   /{resource}/create          # 新建（用 create，禁止 add）
POST   /{resource}/update          # 更新
DELETE /{resource}/delete/{id}     # 删除单条
DELETE /{resource}/batch-delete    # 批量删除（kebab-case）
POST   /{resource}/query           # 分页查询
GET    /{resource}/detail/{id}     # 详情
POST   /{resource}/online/{id}     # 上架（POST，禁止 PUT）
POST   /{resource}/offline/{id}    # 下架（POST，禁止 PUT）
```

路径全部 kebab-case，动态参数统一用 `{id}`（禁止 `{storeId}` 等业务前缀）。

### 前端关键禁忌

- 禁止使用 `Modal.confirm`、`message.success` 等静态方法 → 统一从 `AppProvider` 导入
- 禁止硬编码任何用户可见文案 → 必须走 `t()`
- 禁止独立的 `XxxCreate` / `XxxDetail` 页面组件 → 统一用 `XxxEditor` + `mode` prop
- 禁止自己写页面骨架 → 统一用 `ContentDetailLayout`

### 数据库

- 所有 DB 变更必须写 Flyway 迁移文件（`casalyin-java/.../db/migration/V{n}__xxx.sql`）
- 禁止手动在 DMS 或数据库客户端执行 SQL

---

## 测试账号（本地）

| 角色 | 入口 | 账号 | 密码 | 验证码 |
|------|------|------|------|--------|
| BASE | `/login` | `123456@qq.com` | `123456` | `123456` |
| COMPANY | `/login` | `123123@qq.com` | `123456` | `123456` |
| ADMIN | `/internal/auth-admin-portal` | `custom` | `123123` | 无 |

本地验证码固定为 `123456`（Playwright 直接填入）。

---

## 待开发需求（遗留）

- **Admin 下架通知（CONTENT_OFFLINE）**：Admin 强制下架内容时，向内容所属用户发送系统通知。详见 `docs/decisions/architecture-decisions.md#ADR-005`。

---

## 团队运营（多智能体模式）

> 本节仅在多智能体团队运营时有效。

### 团队成员

| 名称 | 角色 | 核心能力 |
|------|------|---------|
| frontend-dev | 前端开发 | React + TypeScript + Ant Design，遵循 XxxEditor 架构 |
| backend-dev | 后端开发 | Spring Boot + Sa-Token + MyBatis Plus + Flyway |
| e2e-tester | 联调测试 | Playwright 测试，每次代码变更后必须运行验证 |
| reviewer | 代码审查 | 只读审查，安全/质量/项目规范检查 |

### 规划文件位置

```
.plans/casalyin/
  task_plan.md          ← 主计划
  findings.md           ← 团队汇总
  progress.md           ← 工作日志
  decisions.md          ← 架构决策
  frontend-dev/         ← 前端开发智能体工作目录
  backend-dev/          ← 后端开发智能体工作目录
  e2e-tester/           ← 测试智能体工作目录
  reviewer/             ← 代码审查智能体工作目录
  researcher/           ← 研究智能体工作目录（第二阶段）
```

### 核心协议

| 协议 | 规则 |
|------|------|
| 每次代码变更 | e2e-tester 必须运行相关测试脚本确认功能正常 |
| 功能完成 | backend-dev / frontend-dev 直接找 reviewer 做代码审查 |
| 决策点 | 上报 team-lead，由 team-lead 咨询用户后决定 |
| Bug 3 次无法修复 | 上报 team-lead |
| 上下文恢复 | 读 `.plans/casalyin/team-snapshot.md` |
| 测试执行 | **只有 e2e-tester 跑测试**，team-lead 不直接执行测试；需要用户操作（重启服务/准备数据）时由 team-lead 告知用户 |

### Known Pitfalls

- Ant Design Modal/message 静态方法在 `ConfigProvider` 下无任何响应（无报错极难排查）→ 统一从 `AppProvider` 导入
- 路由路径若含 `/admin/` 同时标注 `@NoNeedLogin` 会完全绕过鉴权 → 禁止
- `@SaIgnore` 会绕过 Sa-Token 所有鉴权，公开接口只用 `@NoNeedLogin`
- 草稿 PENDING 状态下后端必须拦截所有写操作（不依赖前端只读 UI）
- BASE 角色权限码须覆盖所有业务维度的草稿读写操作（`product/store/agronomist` 的 draft:create/update/query/submit 等），漏加会导致前端静默失败
- **Flyway 迁移文件一旦执行过，绝对不能修改内容**（会导致 checksum 不一致，Spring 容器初始化中断）→ 需要修改表结构必须新建 Vn+1 文件；开发环境紧急修复可 `UPDATE flyway_schema_history SET checksum = <新checksum> WHERE version = 'n'`

### E2E 测试排查规范（三步顺序）

测试失败时 e2e-tester **必须按此顺序**排查，不得跳步：

1. **页面是否符合预期**（有无跳转、列表有无更新）
2. **页面上有无弹窗/toast**（成功或失败都应有提示；静默失败本身是 Bug，需单独上报）
3. **最后才看 API response**（`waitForResponse` + `console.log` 抓 status 和 body）
