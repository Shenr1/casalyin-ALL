# Casalyin 项目规范

> 本文件会在每次 Claude Code 会话启动时自动加载。
> 规范详情在 `docs/` 目录下，此处只列索引和关键约定。
> 开发计划索引在 `.plans/casalyin/docs/index.md`。

---

## 开发流程

**当前阶段：单人开发，生产环境未对外开放。**

- 功能开发、本地服务启动/调试、测试验证、commit、push 全部在本地完成
- 直接在 `main` 分支开发并 push（主仓库 casalyin-ALL 为 `master`）
- push 后 CI/CD 自动部署到生产环境，因生产未开放，无影响面

生产环境排查、服务器监控、生产库运维、生产密钥配置由 Hermes 负责（本地一律自己来）。

> 等生产正式对外开放后，再重新引入 develop 分支与发版评审流程。

---

## 文档职责分工

**规则：CLAUDE.md 只写规范，不写任务。任务、Bug、待确认问题一律写到对应专属文件。**

| 文档 | 存放内容 |
|------|---------|
| `CLAUDE.md` | 项目规范、约定、禁忌、Known Pitfalls |
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
| `docs/testing/BUSINESS-FLOW-ANALYSIS.md` | 业务流程分析 |

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
| `.plans/casalyin/task_plan.md` | 主计划与遗留需求 |

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

## Git 提交与发布

**当前阶段：直接在 `main` 分支开发并 push（主仓库 casalyin-ALL 为 `master`）。**

- push 后 CI/CD 自动部署到生产环境；生产未对外开放，无影响面
- 不使用 develop / feature / hotfix 分支（单人开发，双分支流程是过度设计）
- 仓库当前无 tag；不做版本封板

**Commit message 格式：** `feat: {简短描述}` / `fix: {简短描述}` / `chore: {简短描述}`

### 子模块提交顺序（重要）

三个业务仓库是 git submodule。改动涉及子模块时必须先提交子模块，再提交主仓库指针，否则主仓库指针会指向不含改动的 commit：

```bash
# 1. 先在子模块内提交并 push
cd casalyin-java && git add <files> && git commit -m "fix: ..." && git push origin main

# 2. 再回主仓库提交指针
cd .. && git add casalyin-java && git commit -m "chore: bump submodule" && git push origin master
```

`git status` 中子模块显示 `Mm` 表示指针已变**且**内部还有未提交内容 —— 此时不要直接提交主仓库。

> 等生产正式对外开放后，再引入 develop 分支与发版评审流程。历史流程记录见 `docs/git-workflow.md`（内容已过时，仅供参考）。

---

## 本地开发环境操作

**完整启动指南：** `docs/development/startup-guide.md`
**启动故障排查：** `docs/development/local-startup-troubleshooting.md`

### 服务启动命令（标准）

| 服务 | 端口 | 启动命令 | 等待时间 |
|------|------|---------|---------|
| 后端 Spring Boot | 8690 | `cd casalyin-java && npm run dev` | ~30-40 秒 |
| 后台前端 Vite | 5173 | `cd casalyin-server && npm run dev` | ~5 秒 |
| C端前台 Next.js | 3000 | `cd casalyin-Headless && npm run dev` | ~10 秒 |

**启动均使用 `run_in_background: true`，启动后等待对应时间再继续操作。**

### 重启服务

```bash
# 后端重启（dev:stop 内部是 lsof -ti :8690 | xargs kill -9）
cd casalyin-java && npm run dev:stop && npm run dev

# 前端重启（先停止占用端口的进程）
lsof -ti :5173 | xargs kill -9 2>/dev/null || true
cd casalyin-server && npm run dev
```

### 强制重新编译（后端）

当修改了 Java 代码但服务没有生效时：

```bash
cd casalyin-java
rm -rf casalyin-admin/target    # 删除编译缓存
npm run dev                      # 重新编译并启动
```

> Maven 会跳过未变更的模块并输出 `Nothing to compile - all classes are up to date`，
> 此时并未真正验证改动，必须删 target 后重编。

### 后端约束

- 开发机为 macOS (Apple Silicon)，JDK 17 由 Homebrew 安装（`brew install openjdk@17`）
- `scripts/dev-admin.sh` 自动检测 `JAVA_HOME`（依次尝试 `/opt/homebrew` 与 `/usr/local` 下的 `openjdk@17`），无需手动配置
- 后端使用 Maven Wrapper（mac 用 `./mvnw`，`mvnw.cmd` 是 Windows 那份），无需安装 Maven
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

## Known Pitfalls（踩过的坑，必读）

### 后台（casalyin-server / casalyin-java）

- Ant Design Modal/message 静态方法在 `ConfigProvider` 下无任何响应（无报错极难排查）→ 统一从 `AppProvider` 导入
- 路由路径若含 `/admin/` 同时标注 `@NoNeedLogin` 会完全绕过鉴权 → 禁止
- `@SaIgnore` 会绕过 Sa-Token 所有鉴权，公开接口只用 `@NoNeedLogin`
- 草稿 PENDING 状态下后端必须拦截所有写操作（不依赖前端只读 UI）
- BASE 角色权限码须覆盖所有业务维度的草稿读写操作（`product/store/agronomist` 的 draft:create/update/query/submit 等），漏加会导致前端静默失败
- **Flyway 迁移文件一旦执行过，绝对不能修改内容**（会导致 checksum 不一致，Spring 容器初始化中断）→ 需要修改表结构必须新建 Vn+1 文件；开发环境紧急修复可 `UPDATE flyway_schema_history SET checksum = <新checksum> WHERE version = 'n'`
- **更新实体禁止用 `SmartBeanUtil.copy` 整体覆盖**：前端未传的字段会被写成 null，导致编辑时其他字段被清空 → 在已查出的实体上按字段合并，只覆盖非 null 字段
- 前端 payload 用 `values.x || undefined` 会让"主动清空"和"未修改"无法区分 → 需要支持清空的字段传 `null`，未修改才传 `undefined`

### C 端前台（casalyin-Headless）

- Next.js App Router 中，Server Component 不能直接使用 useState/useEffect → 需拆分为 Client Component
- 环境变量在 Server Component 中用 `process.env.XXX`，在 Client Component 中必须以 `NEXT_PUBLIC_` 前缀暴露
- API 调用失败时 Next.js Server Component 会直接报错（不是 JSON 错误），需加 try/catch

### C 端质量红线

| 维度 | 要求 |
|------|------|
| 响应式 | 375px 无横向溢出，触控目标 ≥44px，所有功能可用 |
| SEO & 性能 | 优先 Server Components / SSG，图片必须有 alt，LCP < 2.5s |
| 接口一致性 | 字段名与 `.plans/casalyin-headless/docs/api-contracts.md` 一致，路径遵循 kebab-case |

---

## 调试排查顺序（四步，不得跳步）

功能异常或测试失败时按此顺序排查：

0. **先检查环境**（最优先）：
   - 前端服务是否正常（Vite 有无编译错误 overlay、是否存在浏览器/Vite 缓存问题）
   - 后端服务是否已启动并完成初始化（Flyway、缓存预热等）
   - 权限/会话是否是全新状态（是否需要重新登录刷新缓存）
   - **环境问题优先于代码问题排查，避免误判**
1. **页面是否符合预期**（有无跳转、列表有无更新）
2. **页面上有无弹窗/toast**（成功或失败都应有提示；静默失败本身就是 Bug）
3. **最后才看 API response**（`waitForResponse` + `console.log` 抓 status 和 body）

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
