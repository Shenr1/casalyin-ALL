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

## D13: 仓库与物流功能（v2.0 规划中）

- 日期: 2026-04-14
- 状态: 需求讨论中，待用户确认后立项
- 核心模式: 类 JD 自营仓储物流——品牌/店铺把货放你们秘鲁仓库，你们负责发货
- 收费模式: 仓储费 + 物流费 + 广告费，未来扩展佣金/支付/浮动利息
- 参考: 17track.net（物流追踪接口）、JD 自营模式、amz123

### 待用户确认的问题

1. **货物维度关系**：库存挂在「产品」下还是「店铺」下？一个产品对应多个 SKU 库存，还是以店铺为单位管理？

2. **操作角色**：入库/出库由 ADMIN 运营人员操作，还是 COMPANY 品牌方自己提交申请，或两者都有？

3. **物流模式**：自有车辆发货，还是接顺丰/DHL 等三方？运单号是自己生成还是三方返回？17track 用于追踪，是手动录入运单号还是自动同步？

4. **C 端展示**：品牌方/买家能否在 C 端查看货物实时位置？还是只在后台管理系统可见？

5. **优先级**：确认为 v2.0，不插队当前 v1.x 计划。
