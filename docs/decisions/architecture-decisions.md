# 架构决策记录（ADR）

> 最后更新：2026-04-11

---

## ADR-001 Dashboard Admin 视图精简

**日期：** 2026-04-03（已完成开发）

**状态：** ✅ 已落地

### 决策

Admin Dashboard 从 11 个 API 精简为 5 个。只保留「待审核待办」模块，其余全部砍掉。

### 背景

Dashboard 原设计有两个模块：
- 平台数据统计（5 个 API）：用户总数、产品总数、店铺总数、品牌总数、农艺师总数
- 最近注册用户（1 个 API）
- 待审核待办（5 个 API）

### 决策理由

**砍掉的模块：**
- **平台数据统计** — vanity metrics，不驱动操作，Admin 看了不会去做什么事，全部删除
- **最近注册用户** — 有了更好没了无所谓，整块删除

**保留的模块：**
- **待审核待办** — Admin 最核心需求，进来第一眼就要知道有无待办事项

### 实现细节

待审核待办 5 个接口：

| 维度 | 接口 | 说明 |
|------|------|------|
| 产品 | `POST /product/draft/query` | `pageSize:1 + searchCount:true + status:0` 拿 total |
| 店铺 | `POST /store/draft/query` | 同上 |
| 农艺师 | `POST /agronomist/draft/query` | 同上 |
| 品牌草稿 | `GET /brand/draft/pending/count` | **新增 count 接口**（原来拉全量列表取 .length，浪费） |
| 品牌申请 | `GET /brand/apply/pending/count` | **新增 count 接口** |

新增后端接口：
- `GET /brand/draft/pending/count` → `BrandController`，需 `brand:audit` 权限
- `GET /brand/apply/pending/count` → `BrandApplyController`，需 `brand:audit` 权限
- 实现：`selectCount + LambdaQueryWrapper`，不改 DAO

### 原则（适用于未来 Dashboard 扩展）

**新增 Dashboard 模块前必须回答：「这个数字会让 Admin 点进去做什么？」**

不能回答这个问题 → 不加这个模块。

---

## ADR-002 XxxEditor 三合一架构

**日期：** 2026-04-10（已完成开发）

**状态：** ✅ 已落地（四个维度均已完成）

### 决策

四个业务维度（店铺、产品、品牌、农艺师）的新建/草稿编辑/详情页，统一为单一 `XxxEditor` 组件，通过 `mode` prop 区分行为。

**不存在独立的 `XxxCreate` / `XxxDetail` / `XxxDraftEdit` 页面组件。**

### 决策理由

避免 create / edit / detail 三个页面重复维护相同逻辑（表单字段、权限判断、提交流程），保证 UI / 交互在三种模式下完全一致。

### 参见

详细规范见 `docs/frontend/editor-architecture.md`。

---

## ADR-003 Flyway 数据库迁移接入

**日期：** 2026-04-06（已接入）

**状态：** ✅ 已落地

### 决策

接入 Flyway 管理数据库版本，V1 作为基线（来自 DMS 导出），禁止手动执行 SQL。

### 决策理由

之前散落的 `migration_*.sql` 文件无版本顺序，手动执行容易遗漏，导致 Entity 与数据库不同步。

### 参见

详细规范见 `docs/backend/database.md`。

---

## ADR-004 上下架语义统一（disabledFlag）

**日期：** 2026-04-09（进行中）

**状态：** ⏳ 部分完成（产品/品牌已完成，店铺/农艺师前端待统一）

### 决策

四个维度的上下架操作统一用 `disabled_flag` 字段（boolean），不使用 `status` 字段或其他字段做上下架。

上下架是「可见性」控制，不影响数据存在性（不做物理删除）。

### 参见

详细规范见 `docs/business/online-offline.md`。

---

## ADR-005 Admin 离线（下架）通知（待开发）

**日期：** 2026-04-11

**状态：** ⏳ 未开发（遗留需求）

### 决策

当 Admin 对内容执行强制下架操作时，需向内容所属用户发送系统通知（CONTENT_OFFLINE 类型）。

### 背景

当前 Admin 可以强制下架四个维度的内容，但不会向内容所有者发送任何通知，导致用户不知道自己的内容被下架及原因。

### 范围

- 触发条件：Admin 调用 `/admin/{resource}/offline/{id}` 接口
- 通知类型：`CONTENT_OFFLINE`
- 通知接收方：被下架内容的 `user_id` 对应用户
- 通知内容：至少包含内容名称、下架原因（可选）、操作时间
