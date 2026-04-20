# Casalyin 项目规范

> 本文件会在每次 Claude Code 会话启动时自动加载。
> 规范详情在 `docs/` 目录下，此处只列索引和关键约定。
> 开发计划索引在 `.plans/casalyin/docs/index.md`。

---

## 文档职责分工

**规则：CLAUDE.md 只写规范，不写任务。任务、Bug、待确认问题一律写到对应专属文件。**

| 文档 | 存放内容 |
|------|---------|
| `CLAUDE.md` | 项目规范、约定、禁忌、Known Pitfalls、团队运营协议 |
| `.plans/*/task_plan.md` | 任务列表、版本进度、待办事项 |
| `casalyin-server/ISSUES.md` | 已知 Bug 与问题清单 |
| `casalyin-server/PENDING_DISCUSSIONS.md` | 待确认的业务问题 |
| `docs/decisions/architecture-decisions.md` | 架构决策记录（ADR） |

> 遗留需求见 `.plans/casalyin/task_plan.md`；已知问题见 `casalyin-server/ISSUES.md`；待确认问题见 `casalyin-server/PENDING_DISCUSSIONS.md`

---

## 项目结构

```
casalyin/
  casalyin-server/     ← 后台前端（React + Vite + TypeScript + Ant Design）
  casalyin-java/       ← 后端（Spring Boot + Sa-Token + MyBatis Plus + Flyway）
  casalyin-Headless/   ← C 端前台（不在本项目维护范围）
  docs/                ← 项目规范文档（团队必读）
  .plans/casalyin/     ← 开发计划与进度记录
```

---

## 规范索引（docs/）

### 业务规范

| 文件 | 内容 |
|------|------|
| `docs/business/roles-permissions.md` | BASE/COMPANY/ADMIN 权限边界、COMPANY 身份流转、订阅与冻结逻辑 |
| `docs/business/workflow-draft.md` | 草稿状态机（DRAFT→PENDING→发布/拒绝）、免审直发、敏感词检测 |
| `docs/business/online-offline.md` | 上下架语义（disabledFlag）、四维度 API 规范、前端交互规范、运行时 Bug |
| `docs/business/vip-monetization.md` | VIP 体系、各维度 VIP 效果、VipBanner 规范、商业模式 |

### 前端规范

| 文件 | 内容 |
|------|------|
| `docs/frontend/ui-components.md` | 页面宽度、返回按钮、ContentDetailLayout、Table、Modal/message 使用、权限控制 |
| `docs/frontend/i18n.md` | 禁止硬编码文案、t() 规范、key 命名、三语言同步、lint 检查 |
| `docs/frontend/routing.md` | 完整路由清单，新增页面必须先查此表 |
| `docs/frontend/editor-architecture.md` | XxxEditor 三合一架构（mode: create/draft/detail）、三期待办 |
| `casalyin-server/docs/list-page-spec.md` | 列表页统一规范（ListSearchSortBar、Tabs、Table、批量操作） |

### 后端规范

| 文件 | 内容 |
|------|------|
| `docs/backend/api-conventions.md` | HTTP 方法、路径命名、鉴权注解、权限码命名、四维度接口矩阵 |
| `docs/backend/database.md` | Flyway 迁移规范、字段命名约定、核心表结构、COMPANY 订阅表 |

### 测试规范

| 文件 | 内容 |
|------|------|
| `docs/testing/test-strategy.md` | Playwright 规范、测试账号、文件结构、断言规范、测试优先级 |
| `docs/testing/test-cases.md` | 完整测试用例清单（十一期 800+ 行，覆盖状态机/权限/VIP/UI 规范） |
| `test-cases.md` | 早期测试用例汇总（部分与 docs/testing/test-cases.md 重复，以 docs/ 为准） |

### 架构决策

| 文件 | 内容 |
|------|------|
| `docs/decisions/architecture-decisions.md` | ADR 记录：Dashboard 精简、XxxEditor、Flyway、上下架统一、待开发需求 |
| `casalyin-server/ISSUES.md` | 全链路验证问题清单（P1 功能断裂/P2 i18n/P3 代码质量/P4 规范对齐） |
| `casalyin-server/PENDING_DISCUSSIONS.md` | 待确认问题（COMPANY 品牌字段自动填入） |

### Dashboard

| 文件 | 内容 |
|------|------|
| `casalyin-server/docs/DASHBOARD-DEVELOPMENT.md` | Dashboard 管理员工作台 / SHOP/COMPANY 概览 / 快捷入口开发文档 |

### 开发计划

| 文件 | 内容 |
|------|------|
| `.plans/casalyin/docs/index.md` | 知识库索引（新增页面查此入口） |
| `EDITOR_REFACTOR_PLAN.md` | Editor 规范统一四期计划（农艺师✅/店铺进行中/收尾待办） |

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
- 后端 Form/Entity 校验注解必须用 `jakarta.validation.constraints.*`，**禁止用 `javax.validation.constraints.*`**（Docker 构建环境为 Java 17 + Spring Boot 3.x，javax 包已移除，会导致 CI/CD 编译失败）

---

## 版本发布协议

**用户不手动 push。所有 push 由 team-lead 执行，流程如下：**

```
功能完成 → reviewer [OK] + E2E 通过
    ↓
team-lead 向用户汇总本版本内容，请求确认
    ↓
用户确认
    ↓
team-lead 执行：git add → git commit → git push origin main
    ↓
ECS 自动部署
```

**Commit message 格式：** `feat: v{版本号} {简短描述}`
例：`feat: v1.3 brand dashboard + product list`

---

## 本地开发环境操作

**所有服务均可由 agent 通过 Bash 工具启动，无需用户手动操作。**

### 服务启动命令

| 服务 | 端口 | 启动命令 | 等待时间 |
|------|------|---------|---------|
| 后端 Spring Boot | 8690 | `powershell.exe -File "casalyin-java/scripts/dev-admin.ps1"` | ~30 秒 |
| 后台前端 Vite | 5173 | `cd casalyin-server && npm run dev` | ~5 秒 |
| C端前台 Next.js | 3000 | `cd casalyin-Headless && npm run dev` | ~10 秒 |

**启动均使用 `run_in_background: true`，启动后等待对应时间再继续操作。**

### 后端约束

- JDK 17 必须在 `C:\Program Files\Eclipse Adoptium\jdk-17.0.17.10-hotspot`，脚本写死此路径
- 启动后 ADMIN 需重新登录刷新权限缓存（Flyway 变更后必须）

### Redis 缓存问题

若启动后出现 `ClassCastException: JSONArray cannot be cast to UserPermission`：
```bash
redis-cli -n 1 flushdb
```
然后重新登录即可。

---

## 测试账号（本地）

| 角色 | 入口 | 账号 | 密码 | 验证码 |
|------|------|------|------|--------|
| BASE | `/login` | `123456@qq.com` | `123456` | `123456` |
| COMPANY | `/login` | `123123@qq.com` | `123456` | `123456` |
| ADMIN | `/internal/auth-admin-portal` | `custom` | `123123` | 无 |

本地验证码固定为 `123456`（Playwright 直接填入）。

---

## 多智能体模式

### 团队命名与上下文恢复

**团队名格式：`casalyin-YYYYMMDD`**（例：`casalyin-20260414`）

当前团队名存储在 `.plans/casalyin/current-team.txt`。

**每次 session 启动（含上下文重置后）必须执行：**
1. 读 `.plans/casalyin/current-team.txt` 拿到当前团队名
2. 尝试 `SendMessage` 唤醒已有 agent，**不要直接 spawn 新的**
3. 若 agent 无响应（已退出），再在同一团队名下 spawn 新 agent
4. 跨天时用新日期建新团队，并更新 `current-team.txt`

**禁止在没有检查 `current-team.txt` 的情况下直接 spawn agent** → 这是产生重复 agent 的根本原因。

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
  progress.md           ← 工作日志
  decisions.md          ← 架构决策
  current-team.txt      ← 当前团队名（session 恢复入口）
```

### 核心协议

| 协议 | 规则 |
|------|------|
| 每次代码变更 | e2e-tester 必须运行相关测试脚本确认功能正常 |
| 功能完成 | frontend-dev / backend-dev 完成后找 reviewer 审查 |
| 决策点 | 上报 team-lead，由 team-lead 咨询用户后决定 |
| Bug 3 次无法修复 | 上报 team-lead |
| 上下文恢复 | 读 `current-team.txt` → SendMessage 唤醒 → 读 `task_plan.md` 恢复进度 |
| 测试执行 | **只有 e2e-tester 跑测试**，team-lead 不直接执行测试；需要用户操作时由 team-lead 告知用户 |

### Known Pitfalls

- Ant Design Modal/message 静态方法在 `ConfigProvider` 下无任何响应（无报错极难排查）→ 统一从 `AppProvider` 导入
- 路由路径若含 `/admin/` 同时标注 `@NoNeedLogin` 会完全绕过鉴权 → 禁止
- `@SaIgnore` 会绕过 Sa-Token 所有鉴权，公开接口只用 `@NoNeedLogin`
- 草稿 PENDING 状态下后端必须拦截所有写操作（不依赖前端只读 UI）
- BASE 角色权限码须覆盖所有业务维度的草稿读写操作（`product/store/agronomist` 的 draft:create/update/query/submit 等），漏加会导致前端静默失败
- **Flyway 迁移文件一旦执行过，绝对不能修改内容**（会导致 checksum 不一致，Spring 容器初始化中断）→ 需要修改表结构必须新建 Vn+1 文件；开发环境紧急修复可 `UPDATE flyway_schema_history SET checksum = <新checksum> WHERE version = 'n'`

### E2E 测试排查规范（四步顺序）

测试失败时 e2e-tester **必须按此顺序**排查，不得跳步：

0. **先检查环境**（最优先，排查前置条件）：
   - 前端服务是否正常（Vite 有无编译错误 overlay、是否存在浏览器/Vite 缓存问题）
   - 后端服务是否已启动并完成初始化（Flyway、缓存预热等）
   - 权限/会话是否是全新状态（browserContext 是否已隔离、是否需要重新登录刷新缓存）
   - **环境问题优先于代码问题上报，避免误判**
1. **页面是否符合预期**（有无跳转、列表有无更新）
2. **页面上有无弹窗/toast**（成功或失败都应有提示；静默失败本身是 Bug，需单独上报）
3. **最后才看 API response**（`waitForResponse` + `console.log` 抓 status 和 body）

---

## casalyin-headless 团队运营手册

> C 端前台团队（casalyin-Headless/）。由 CCteam-creator-cn 于 2026-04-19 生成。

### Team-Lead 控制平面

- team-lead = 主对话，不是生成的 agent
- **禁用独立子智能体**：团队存在后，所有工作通过 SendMessage 交给队友

### 团队花名册

| 名称 | 角色 | 模型 | 核心能力 |
|------|------|------|---------|
| frontend-dev | 前端开发 | sonnet | Next.js 14 + TypeScript + Tailwind，Server Components，响应式 |
| e2e-tester | 联调测试 | sonnet | Playwright 测试 + 浏览器自动化 |
| reviewer | 代码审查 | sonnet | 安全/质量/性能/规范审查（只读） |

### 团队名称与恢复

**团队名格式：`casalyin-headless-YYYYMMDD`**（例：`casalyin-headless-20260419`）

当前团队名存储在 `.plans/casalyin-headless/current-team.txt`。

**每次 session 启动必须执行：**
1. 读 `.plans/casalyin-headless/current-team.txt` 拿到当前团队名
2. 尝试 `SendMessage` 唤醒已有 agent，**不要直接 spawn 新的**
3. 若 agent 无响应（已退出），再在同一团队名下 spawn 新 agent

### 规划文件位置

```
.plans/casalyin-headless/
  task_plan.md          ← 主计划
  progress.md           ← 工作日志
  decisions.md          ← 架构决策
  current-team.txt      ← 当前团队名（session 恢复入口）
  docs/                 ← 知识库（architecture, api-contracts, invariants, index）
  frontend-dev/         ← 前端开发工作目录
  e2e-tester/           ← 测试工作目录
  reviewer/             ← 审查工作目录
```

### 通信速查

| 操作 | 命令 |
|------|------|
| 给前端分配任务 | `SendMessage(to: "frontend-dev", message: "...")` |
| 给测试分配任务 | `SendMessage(to: "e2e-tester", message: "...")` |
| dev 请求代码审查 | frontend-dev 直接联系 reviewer（不经过 team-lead） |

### 审查维度

| # | 维度 | 权重 | STRONG 表现 | WEAK 表现 |
|---|------|------|------------|---------|
| RD-1 | 响应式/移动端适配 | 高 | 375px 无横向溢出，触控目标 ≥44px，所有功能可用 | 移动端溢出/截断，按钮太小无法点击 |
| RD-2 | SEO & 性能 | 高 | Server Components / SSG，图片有 alt，LCP<2.5s | 全客户端渲染，无 alt 属性，首屏白屏 |
| RD-3 | 业务规范一致性 | 中 | API 字段名与 api-contracts.md 一致，遵循 kebab-case 路径规范 | 自己猜测字段名，路径命名不规范 |

### headless 团队 Known Pitfalls

- Next.js App Router 中，Server Component 不能直接使用 useState/useEffect → 需拆分为 Client Component
- 环境变量在 Server Component 中用 `process.env.XXX`，在 Client Component 中必须以 `NEXT_PUBLIC_` 前缀暴露
- API 调用失败时 Next.js Server Component 会直接报错（不是 JSON 错误），需加 try/catch

---

# Karpathy Coding Principles

## 1. Think Before Coding
Surface uncertainty early. "If multiple interpretations exist, present them - don't pick silently."

## 2. Simplicity First
Write the minimum viable solution. Ask: *"Would a senior engineer say this is overcomplicated?"*

## 3. Surgical Changes
Modify only what's necessary. "Every changed line should trace directly to the user's request."

## 4. Goal-Driven Execution
Turn vague tasks into verifiable outcomes — e.g., write a failing test first, then make it pass. Use a brief numbered plan for multi-step work.

---

**Core tradeoff:** These rules favor caution over speed — use judgment on trivial tasks.

**Success looks like:** fewer unnecessary diff changes, less over-engineering, and clarifying questions *before* mistakes rather than after.
