# casalyin - 架构决策记录（团队级）

> 团队运营决策。项目架构决策见 docs/decisions/architecture-decisions.md。

---

## D1: 团队角色配置

- 日期: 2026-04-11
- 决策: frontend-dev + backend-dev + e2e-tester + reviewer，暂不启动 researcher 和 custodian
- 理由: 当前任务是 Bug 修复，无需大量调研；custodian 适合 4+ 人大团队
- researcher 启动时机: 阶段 1（Bug 清单）修复完成后

## D2: 工作顺序

- 日期: 2026-04-11
- 决策: 先处理用户提供的 Bug 清单，再由 researcher 做全面扫描
- 理由: 用户明确表示有已知 Bug 清单，优先解决已知问题

## D3: BASE 角色全局搜索权限

- 日期: 2026-04-11
- 决策: 将 `search:query` 加入 `BASE_PERMISSION_CODES`，全局搜索对所有登录用户开放
- 理由: 用户确认搜索功能无需限制角色
- 执行: backend-dev 修改 PermissionService.java

## D10: COMPANY 用户 SOFT 违规词处理

- 日期: 2026-04-12
- 决策: COMPANY 角色命中 SOFT 违规词时，也直接发布（不转人工审核）
- 理由: D6"免审直发"包含跳过 SOFT 违规词检测；COMPANY 是付费认证品牌方，内容质量有保障
- 影响: workflow-draft.md 已更新注明 COMPANY 例外；StoreDraftService 误导性注释已修正

## 待开发 Backlog

| # | 内容 | 优先级 |
|---|------|--------|
| B1 | 产品维度 submitDraft 缺少违规词检测（遗留 Bug） | 中 |
| B2 | `isCompanyUser()` 方法在三个 Service 重复，抽离为 `RoleUtils` 工具类 | 低 |
| B3 | D9 平台认证申请体系 + ContentStatusCard 四维度统一组件 | 中 |
| B4 | i18n 补全：`productForm.brand` / `storeForm.fields.brand` 缺 en/es-ES 翻译（存量问题）| 低 |

## D9: 平台认证申请体系

- 日期: 2026-04-12
- 更新: 2026-04-12（分期规划 + Admin 审核入口决策）
- 决策: 用户可主动申请平台认证，Admin 审核通过后开启认证标记
- 规则:
  - 申请人：BASE 和 COMPANY 均可申请
  - 费用：免费（付费通道尚未上线，现阶段联系我们付费）
  - 拒绝原因：**非必填**（恶意提交无需给理由）
  - 冷却期：拒绝后 3 天内不可再申请
  - 申请状态：PENDING / APPROVED / REJECTED
- 后台引导 UI：内容详情页显示三个状态（品牌认证/平台认证/VIP），未获得的显示"申请认证"入口
  - 必须四个维度统一，抽离为共享组件（暂定名 ContentStatusCard）
- 开发顺序：品牌认证体系（D5~D8）完成后再开发此功能

### 分期规划（B3）

**期一：后端（数据层 + 全部接口）**
- Flyway V5：新建 `t_certification_apply` 表
  - 字段：id / resource_type / resource_id / applicant_id / status / reject_reason（nullable）/ rejected_at / created_at / updated_at
- 用户接口：发起申请 / 取消申请 / 查询当前资源的申请状态
- Admin 接口：审核通过 / 审核拒绝（拒绝原因非必填）
- 业务规则：冷却期 3 天校验、重复申请拦截（已有 PENDING 不能再申请）

**期二：前端 ContentStatusCard + 申请流程**
- 新建共享组件 `ContentStatusCard`，四个维度 Editor 详情页侧边栏统一接入
- 用户视角：三状态展示 + 申请/取消按钮 + 冷却期提示
- Admin 视角：有 PENDING 申请时显示「通过/拒绝」操作（替代现有直接操作按钮）

**期三：Admin 审核入口（Dashboard + 通知）**
- Admin 审核时触发站内通知（走现有通知体系，同 CONTENT_OFFLINE 机制）
- Dashboard 加「待处理事项」数量提示（认证申请 + 草稿审核合并）
- 点击通知或 Dashboard 数字 → 跳转到对应内容详情页，在 ContentStatusCard 里操作
- **不单独建管理列表页**

## D8: 品牌关联体系（四维度统一）

- 日期: 2026-04-12
- 决策: 所有维度统一用 brand_id 字段表示品牌归属
  - COMPANY 创建任何内容（产品/店铺/农艺师）→ brand_id 自动锁定为其认领品牌，前端只读
  - BASE 创建内容 → brand_id 为空，不允许声称品牌关联（方案 X）
- 前端展示: COMPANY 内容显示"品牌认证"标记，与 BASE 内容形成视觉区分
- 影响: t_store、t_agronomist 表新增 brand_id 字段（Flyway 迁移）；产品已有 brand_id
- 免审扩展: COMPANY 免审直发适用于全部维度（产品/店铺/农艺师）

## D5: 品牌产品归属 — 方案 A

- 日期: 2026-04-12
- 决策: COMPANY 认领品牌后，该品牌下所有产品归 COMPANY 管理（强所有权）
- 理由: BASE 用户主要场景是注册店铺/绑定产品/新增农艺师，品牌内容的主创者是 COMPANY
- 实现: 产品表新增 `brandOwnerId` 字段，creatorId 保留原始创建者，brandOwnerId 表示当前品牌管理者；认领时批量写入 brandOwnerId

## D6: COMPANY 免审直发

- 日期: 2026-04-12
- 决策: COMPANY 提交草稿时后端判断角色，直接跳过 PENDING 发布，不走人工审核
- 理由: COMPANY 是经过认证的品牌方，内容质量有保障
- 实现: 保留草稿流程（create→submit），submit 时后端判断 creatorType == COMPANY → 直接发布；前端 Editor 无需改动
- 附加: COMPANY 创建产品时品牌字段自动锁定为其认领品牌（前端只读）

## D7: 产品归属字段设计

- 日期: 2026-04-12
- 决策: 新增 `brandOwnerId` 字段，与 `creatorId` 分离
- 理由: creatorId 保留历史记录（谁创建的），brandOwnerId 表示品牌管理权（认领后属于谁），审计日志清晰

## D4: 农艺师统一走草稿审核流程

- 日期: 2026-04-11
- 决策: 农艺师与产品/店铺/品牌保持一致，走完整草稿状态机（DRAFT→PENDING→发布/拒绝）
- 理由: 四个维度应统一规范，农艺师目前直接创建/更新的方式与其他维度不一致
- 影响范围:
  - 后端: 新增 t_agronomist_draft 草稿表、draft/create、draft/submit、draft/cancel 等接口、@SaCheckPermission("agronomist:edit") 注解
  - 前端: AgronomistEditor 增加草稿模式（mode: draft），未发布 tab 支持
- 执行: 待规划，backend-dev + frontend-dev 联合实现

## D11: 品牌流量数据采集（规划中）

- 日期: 2026-04-13
- 状态: 规划中（未开发）
- 决策: COMPANY 品牌管理页预留流量数据展示区域（浏览量/点击量等），当前显示占位
- 理由: 流量是品牌方最关心的核心指标，需要尽早规划采集方案
- 后续: 需单独立项，确定采集维度（产品/店铺/农艺师页面的 PV/UV）后再开发

## D12: 同步产品归属 — 自动触发 + 进度展示

- 日期: 2026-04-13
- 状态: 规划中（排入 v1.4 或独立小版本）
- 背景: 当前「同步产品创建者」是产品列表页的一个下拉按钮，位置不合理
- 决策方向:
  - COMPANY 审批通过后，后端自动触发产品归属同步，无需用户手动点击
  - 前端在升级成功的引导页/通知中展示进度条，直观告知品牌方已完成归属转变
  - 产品列表页的手动触发按钮可移除
- 待确认: 具体位置/交互方案待与产品经理对齐后定案
- 影响范围: BrandApplyService.approve() 触发时机、前端引导页/通知设计

## D14: Headless 团队角色规范（2026-04-14）

### headless-dev

**技术栈**：Next.js 14 App Router + React 18 + Tailwind CSS + 原生 fetch + TypeScript 5

**职责**：
- 开发/维护 C 端页面组件（`components/`、`app/` 路由页面）
- 维护 Route Handler BFF 代理层（`app/api/`）
- 维护 `lib/*-api.ts` 客户端接口封装
- i18n 中西双语同步（`lib/i18n/translations/zh.json` + `es.json`）
- 本地自检：读代码 → 跑起来 → 检查效果

**禁忌**：
- 禁止在组件里直接 fetch 真实后端地址 → 必须走 `/api/` Route Handler
- 禁止硬编码用户可见文案 → 必须走 `useTranslation()`
- 禁止在 Route Handler 外暴露 `NEXT_PUBLIC_API_BASE_URL`（只在 Route Handler 内用）

**本地环境**：
- `.env.local` 已配置：`NEXT_PUBLIC_API_BASE_URL=http://localhost:8690`
- Route Handler 内拼 `/api/...` 前缀（base URL 本身不含 `/api`）
- 前端运行：`localhost:3000`，后端：`localhost:8690`

### headless-tester

**技术栈**：Playwright（API 模式为主，UI 模式按需）

**测试位置**：`casalyin-server/e2e/headless/`（复用现有 Playwright 基础设施）

**测试范围**：
- **第一阶段：C 端 API 健康检查（浅度）**
  - 目标：验证各 Route Handler 能正常代理到本地后端
  - 深度：HTTP 200 + 返回 JSON（`code === 0`）即通过，不校验字段内容
  - 覆盖域：product、brand、agronomist、crop、pest、search 等核心 Route Handler
- **第二阶段：UI 功能验证**
  - ProductForumLinks 组件三态（loading → 空状态/列表）
  - 其他功能组件按需

**约定**：
- 测试打 `localhost:3000`（Headless 前端）
- 若 Route Handler 代理失败（后端未启动），测试应明确报 FAIL 而非超时
- API 健康检查用 `request` 模式（无头，不开浏览器），速度快

### 环境依赖
- 本地后端（`localhost:8690`）必须启动
- Headless 前端（`localhost:3000`）必须启动（`npm run dev` 在 casalyin-Headless/）
- `.env.local` 已就位

## D15: 版本管理与上下文清理协议（2026-04-15）

- 日期: 2026-04-15
- 决策: 建立版本粒度规范 + 上下文清理节奏

### 版本粒度规则

| 规则 | 说明 |
|------|------|
| 每个版本完成后，用户执行 `/compact` 清理上下文 | 保持每个 session 上下文干净 |
| 版本不能太大 | 单版本任务超过 8 个（或跨越超过 2 个业务域），必须拆分为 v1.x.1 + v1.x.2 |
| team-lead 负责版本拆分 | 用户不需要自己判断，team-lead 在 session 开始时主动规划并告知 |
| 版本关闭标准 | 所有 E2E 通过 + task_plan.md 标注 ✅ DONE + progress.md 更新 |

### 版本号规则

- `v1.x` — 主版本，包含一组完整功能
- `v1.x.1 / v1.x.2` — 拆分子版本，各自独立关闭，分别执行 `/compact`
- 跨天新 session 必须更新 `current-team.txt`

---

## D16: v1.5 D-论坛3 跳过（2026-04-16）

- 日期: 2026-04-16
- 决策: 跳过 D-论坛3（品牌/店铺/农艺师关联论坛帖子）
- 理由:
  - 当前内容稀缺，论坛帖子主要以产品为中心
  - 将有限帖子分散到 4 个入口（产品/品牌/店铺/农艺师）反而降低每个入口的可用性
  - 产品详情页已有真实 Discourse 帖子展示（D-论坛2），验证了方向正确
- 后续: 等内容积累到一定量级后（如论坛帖子超过 500 条），再重新评估

## D13: 仓库与物流功能（v2.0 规划中）

- 日期: 2026-04-14
- 更新: 2026-04-19（业务模式澄清 + MVP 方案确认）
- 状态: 需求讨论中，不插队当前 v1.x 计划
- 目标市场: **秘鲁**（沟通工具用 WhatsApp / 邮件 / SMS，不用微信）

### 商业模式

**代销返点模式**：品牌方把货放 Casalyin 秘鲁仓库 → Casalyin 帮卖 → 抽返点。

这是平台核心竞争力：品牌不需要自建秘鲁仓、不需要找分销商，Casalyin 提供仓储+销售+配送一站式。

### 完整价值链

```
品牌方把货交给 Casalyin（线下签约）
        ↓
Admin 在后台录入产品 + 可购城市 + 库存数量
        ↓
C端农民看到产品 → 发起聊天（WhatsApp/系统外）
        ↓
Casalyin 告知可购城市和价格 → 农民付款（到付或线上）→ 成交
        ↓
Admin 后台"录入销售"：产品/数量/城市/价格/买家联系方式
        ↓
系统生成订单号，自动发 SMS/邮件给农民（"24小时内收到订单号"）
        ↓
Admin 填入快递单号
        ↓
农民凭订单号来 C 端查询物流（跳转 17track）
        ↓
月结：Admin 导出销售报表 → 线下结算给品牌方（WhatsApp/邮件）
```

**关键原则：聊天、砍价、收钱均在系统外完成；系统只负责记录结果和发订单号。**

### 已确认的 MVP 方案

| 功能 | 方案 | 开发量 |
|------|------|--------|
| 可购城市 | 产品新增 `available_cities` JSON 字段，Admin 手动维护 | 小 |
| 库存管理 | 产品新增 `stock_quantity` 字段，Admin 手动录入/扣减 | 小 |
| Admin 录入销售 | 填交易信息 → 系统生成订单号 → 触发 SMS/邮件通知买家 | 中 |
| 订单追踪（C端） | 输入订单号 → 显示基本信息 + 跳转 17track | 小 |
| 供应商报告 | **不做系统**，Admin 手动导出/发送（WhatsApp/邮件）| 零 |
| 购买意向表单 | **不做**，农民直接 WhatsApp 聊天 | 零 |

### 不做的功能（前期）

- COMPANY 供应商查询门户（前期线下报告替代）
- 自动库存预警（人工盘点）
- 退货退款流程（线下处理）
- 17track API 内嵌（跳转链接即可）

### 品牌方信任建设（内容，无需开发）

- 静态落地页，说明合作流程（发货→代销→月结返点）
- 已合作品牌 logo 墙
- 仓库照片/位置信息

### 待确认问题

| # | 问题 |
|---|------|
| Q5 | 库存是否要 SKU 粒度，还是产品级别够用？ |
| Q9 | 物流状态几档（已出库/运输中/已签收）？ |
| Q11 | 仓库分区/库位管理是否需要？ |
| Q12 | 退货退款如何处理？ |

### D13 与 ADR-005 的关系

仓库功能立项后，CONTENT_OFFLINE 通知（ADR-005）可能需要扩展：
- 产品被下架（disabledFlag=1）时，同步更新库存状态（预留设计空间）
