# 角色体系与权限规则

> 合并自：business_roles.md + business_subscription.md
> 最后更新：2026-04-11

---

## 术语约定

| 说法 | 含义 |
|------|------|
| 后台 | casalyin-server（React 前端管理后台）|
| 后端 | casalyin-java（Spring Boot API）|
| 前台 | headless（面向 C 端用户，另一项目）|

---

## 三种角色

| 角色 | roleCode | 说明 |
|------|----------|------|
| 基础用户 | BASE | 所有注册用户的默认角色 |
| 品牌方 | COMPANY | BASE 升级而来（原账号升级，非新建账号）|
| 管理员 | ADMIN | 最高权限 |

> `UserTypeEnum` 后端只有 `USER` 和 `ADMIN` 两种，**没有 COMPANY**；COMPANY 是 roleCode，不是 UserType。

---

## 四个业务维度

产品 / 品牌 / 店铺 / 农艺师

---

## BASE 权限

- CRUD **自己创建的**产品、店铺、农艺师
- 品牌：只能通过下拉列表选择（只看到 brandId + 名称），无法查看品牌详情，无法创建品牌
- 下架/上架自己的产品、店铺、农艺师（`disabledFlag` 字段）
- **不能删除任何已发布数据**，只能下架
- 草稿只能删除 `draftStatus=0`（未提交）的，PENDING/REJECTED 不可删

---

## COMPANY 权限（基于 BASE 之上）

- B→C 升级：上传营业执照等，走审批流，ADMIN 审批通过后升级；升级前弹出法律条款确认弹窗
- 升级后绑定一个品牌 id（品牌认领）
- 可触发"同步产品"：把所有关联该品牌 id 的产品的 userId 更新为自己；同步后原 BASE 用户无法再 CRUD 这些产品
- 查看产品范围：自己创建的产品 + 认领品牌下所有关联产品（合并展示，按 brandId 过滤）
- 查看店铺范围：仅自己 ownerUserId 名下的店铺
- 可创建新品牌（走审批流程）
- COMPANY 创建的产品、店铺、品牌自动标记 `isCertified=true`
- 下架/上架自己品牌下的产品、店铺、农艺师
- **不能删除任何已发布数据**，只能下架
- **订阅到期后写操作冻结**（草稿 create 均拦截），但角色不变、历史数据保留

---

## COMPANY 身份流转规则

| 操作 | 唯一路径 | 禁止路径 |
|------|----------|----------|
| BASE → COMPANY | 用户提交品牌申请 → Admin 审批通过 | Admin 角色管理页直接授予 ❌ |
| COMPANY → BASE | Admin 在品牌申请详情页"撤销品牌认证" | Admin 角色管理页移除 COMPANY 角色 ❌ |
| 订阅冻结/解冻 | Admin 在品牌申请详情页设置到期时间或手动切换状态 | — |

**后端保护**：`RoleUserService.updateUserRoles()` 凡涉及 COMPANY 角色（授予或移除）一律返回错误。

---

## COMPANY 订阅体系

**Why：** COMPANY 身份以月租形式付费，停止付费后需要冻结写操作但保留角色和历史数据，方便续费后自动恢复。

### 数据存储

订阅状态存在 `t_brand_apply` 表（每个 COMPANY 用户对应一条 `status=1` 的品牌申请记录）：

| 字段 | 类型 | 说明 |
|------|------|------|
| `subscription_expired_at` | datetime NULL | 订阅到期时间，null = 长期有效 |
| `subscription_status` | tinyint DEFAULT 0 | 0-正常 1-手动冻结 |

`t_brand_apply.status` 枚举：0-待审核 / 1-已批准 / 2-已拒绝 / 3-已撤销

### 冻结判断逻辑

满足以下任一条件即为冻结：
```
subscription_status = 1（手动冻结）
OR subscription_expired_at < NOW()（到期）
```

封装在 `BrandApplyService.checkCompanySubscription(userId)`，返回 null = 正常，返回 ResponseDTO = 已冻结。**非 COMPANY 用户调用直接返回 null，不影响 BASE/ADMIN。**

### 冻结校验覆盖范围（已落地，2026-04-03）

四个维度的草稿 create 方法均已接入校验：
- `ProductDraftService.createDraft()`（两个重载均已覆盖）
- `StoreDraftService.createDraft()` 和 `createStoreDraft()`
- `BrandDraftService.addDraft()`
- `AgronomistDraftService.addDraft()`

### Admin 管理入口

`BrandApplyDetail.tsx`（品牌申请详情页）— 仅 status=1（已批准）时展示"订阅管理" Card：
- DatePicker 设置到期时间
- Switch 手动冻结/解冻
- 红色"撤销品牌认证"按钮（二次确认弹窗）

### 相关接口

| 接口 | 说明 |
|------|------|
| `GET /admin/brand-apply/subscription/{userId}` | 查询订阅信息 |
| `POST /admin/brand-apply/subscription/update` | 更新到期时间和冻结状态 |
| `POST /admin/brand-apply/revoke/{userId}` | 撤销品牌认证（移除 COMPANY 角色 + status 改为 3）|

撤销后会返回警告信息，提示 Admin 手动处理品牌归属和关联产品（系统不自动回滚）。

---

## ADMIN 权限

- CRUD 所有维度所有数据，不走草稿审批流程（直接写主表）
- 审批所有草稿（产品/品牌/店铺/农艺师）及 B→C 升级申请
- 查看草稿审批详情时只读，只能通过/拒绝，不能修改草稿内容
- 下架 + 物理删除所有维度所有数据
- 可针对每个用户、每个维度配置数量上限
- 在品牌申请详情页管理 COMPANY 用户订阅（到期时间、冻结状态、撤销认证）

---

## 下架 vs 删除规则

| 操作 | BASE | COMPANY | ADMIN | 字段 |
|------|------|---------|-------|------|
| 下架 | 自己的 | 自己品牌的 | 所有 | `disabledFlag=true` |
| 上架 | 自己的 | 自己品牌的 | 所有 | `disabledFlag=false` |
| 删除（主表已发布数据） | ❌ | ❌ | 所有 | 物理删除 |
| 删除（草稿，仅 DRAFT=0） | 自己的 | 自己的 | 任意状态 | 草稿表物理删除 |

适用维度：产品 ✅、店铺 ✅、农艺师 ✅、品牌 ❌（品牌无上下架）

---

## isCertified 认证字段

| 维度 | 状态 |
|------|------|
| 产品 | 有 `isCertified` ✅ |
| 店铺 | 有 `isCertified` ✅ |
| 品牌 | 有 `isCertified` ✅ |

- COMPANY 创建的内容自动 `isCertified=true`
- ADMIN/BASE 创建的内容默认 `isCertified=false`
- **ADMIN 可手动将任意内容标记为已认证**（在详情页操作）
- BASE 没有认证操作权限

---

## 品牌下拉接口

BASE/COMPANY 选择品牌时调用 `/brand/public/listAll`，登录即可访问，不依赖任何权限码。
