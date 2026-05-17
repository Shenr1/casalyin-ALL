# Casalyin E2E 测试业务流程梳理报告

> 生成时间：2026-05-08
> 目标：建立完整的 E2E 测试规范，核心验证"C端展示 === 后台配置"

---

## 一、测试目标与范围

### 1.1 核心验证目标

**C端展示 === 后台配置**

- 后台配置的所有字段必须在 C端正确展示
- 后台的上下架操作必须在 C端立即生效
- 后台的草稿审核流程必须正确反映在 C端
- VIP 和认证标识必须在 C端正确显示

### 1.2 测试范围

**四个业务维度（按优先级排序）：**

1. **产品（Product）** - 最高优先级
2. **店铺（Store）**
3. **农艺师（Agronomist）**
4. **品牌（Brand）**

**三种角色（按优先级排序）：**

1. **ADMIN** - 管理员操作
2. **COMPANY** - 品牌方操作
3. **BASE** - 基础用户操作

### 1.3 测试环境

- **本地环境**：localhost:3000（C端）、localhost:5173（后台）、localhost:8690（后端）
- **验证码**：固定为 `123456`
- **测试账号**：
  - BASE: 由测试脚本自动创建
  - COMPANY: 由测试脚本从 BASE 升级
  - ADMIN: `custom` / `123123`（登录 URL: `/internal/auth-admin-portal`）

---

## 二、业务规则梳理

### 2.1 角色权限体系

#### BASE 用户权限

- 创建产品、店铺、农艺师（走草稿审批流程）
- 只能 CRUD 自己创建的内容
- 品牌：只能从下拉列表选择，无法查看详情，无法创建
- 下架/上架自己的内容（`disabledFlag` 字段）
- **不能删除已发布数据**，只能下架
- 草稿只能删除 `draftStatus=0`（未提交）的

#### COMPANY 用户权限（基于 BASE 之上）

- BASE 升级而来（上传营业执照 → ADMIN 审批）
- 升级后绑定一个品牌 ID
- 可触发"同步产品"：把关联品牌的所有产品归属到自己
- 查看产品范围：自己创建的 + 认领品牌下的所有产品
- 可创建新品牌（走审批流程）
- COMPANY 创建的内容自动标记 `isCertified=true`
- **订阅到期后写操作冻结**（草稿 create 均拦截），但角色不变

#### ADMIN 权限

- CRUD 所有维度所有数据，不走草稿审批流程（直接写主表）
- 审批所有草稿（产品/品牌/店铺/农艺师）及 B→C 升级申请
- 查看草稿审批详情时只读，只能通过/拒绝
- 下架 + 物理删除所有维度所有数据
- 管理 COMPANY 用户订阅（到期时间、冻结状态、撤销认证）

### 2.2 草稿审核流程

#### 状态流转

```
创建 → [DRAFT 草稿] → 用户提交 → [PENDING 待审核] → admin 通过 → 写入主表（已发布）
                                                     ↘ admin 拒绝 → [REJECTED 已拒绝] → 用户可修改重新提交
```

#### draftStatus 枚举值

| 值 | 枚举 | 含义 |
|----|------|------|
| 0 | DRAFT | 草稿，已保存未提交 |
| 1 | PENDING | 待审核，已提交等待 admin 操作 |
| 2 | REJECTED | 已拒绝，可继续修改再提交 |

#### 编辑已发布内容流程

```
用户点击已发布内容 → 进入详情页（可编辑）
  ↓
用户修改后点击"保存"
  ↓
检查是否已有草稿
  ├─ 没有草稿：创建草稿（复制主表数据 + 用户修改）
  └─ 有草稿：更新草稿
  ↓
draftStatus = 0（草稿状态）
主表不变
```

#### 免审直发规则（品牌/店铺/农艺师）

- **产品维度**：保持原有审核流程不变
- **品牌/店铺/农艺师**：免审直发（除非触发违禁词）

违规词检测分支：
```
submitDraft()
  ├─ 命中 HARD 词 → 返回错误提示，草稿保留
  ├─ 命中 SOFT 词 → COMPANY 角色例外，直接发布；其他角色转人工审核
  └─ 无命中      → 免审直发（写主表 + 记内容变更日志 + 删草稿）
```

### 2.3 上下架规则

#### 业务语义

- **下架**：设置 `disabledFlag = true`，内容仍在主表中，但前台不显示
- **上架**：设置 `disabledFlag = false`，内容在主表中且前台可见
- **品牌维度**：无上下架功能（品牌是用户身份标识，不是内容）

#### 数据存储规则

| 状态 | 存储位置 | 表字段 | 说明 |
|------|---------|--------|------|
| 已发布（上架） | 主表 | `status = 1, disabledFlag = false` | 已发布且上架 |
| 已发布（下架） | 主表 | `status = 1, disabledFlag = true` | 已发布但下架 |
| 未发布（草稿） | 草稿表 | `draftStatus = 0` | 草稿，已保存未提交 |
| 未发布（待审核） | 草稿表 | `draftStatus = 1` | 待审核 |
| 未发布（已拒绝） | 草稿表 | `draftStatus = 2` | 已拒绝 |

**关键规则：**
- 主表保留所有已发布的内容（包括上架和下架的）
- 下架操作只设置 `disabledFlag = true`，不移动到草稿表，不删除主表记录
- C端查询时只显示 `disabledFlag = false` 的内容

### 2.4 VIP 体系

#### 核心概念

- VIP 分配给**内容**（产品/品牌/店铺/农艺师），不是分配给用户账号
- 用户可以单独为某个内容付费，使其成为 VIP 内容
- **核心逻辑：有 VIP = 有曝光入口；无 VIP = 内容存在但用户找不到（URL 仍可访问）**

#### 各维度 VIP 效果

| 维度 | VIP 效果 | 无 VIP |
|------|---------|--------|
| 产品 | 搜索结果优先展示 | 正常排序展示 |
| 店铺 | VIP 店铺才出现在产品详情页「Where to Buy」 | 无入口，URL 直接访问仍可达 |
| 农艺师 | 出现在产品/农作/虫害的「推荐农艺师」模块中 | 无推荐入口，URL 直接访问仍可达 |
| 品牌 | 特殊展示位（待细化）| 正常展示 |

### 2.5 认证标识（isCertified）

| 维度 | 状态 |
|------|------|
| 产品 | 有 `isCertified` ✅ |
| 店铺 | 有 `isCertified` ✅ |
| 品牌 | 有 `isCertified` ✅ |

- COMPANY 创建的内容自动 `isCertified=true`
- ADMIN/BASE 创建的内容默认 `isCertified=false`
- **ADMIN 可手动将任意内容标记为已认证**

---

## 三、完整业务流程（按角色）

### 3.1 ADMIN 操作流程

#### 产品维度

1. **创建产品**
   - 后台：`/admin/product/create` → 直接写主表（不走草稿）
   - C端验证：立即在 `/productos` 列表中显示
   - 关键字段：productName, sku, brandName, pictures, minPrice, maxPrice, info, description, compositions, tags, isCertified, vipLevel

2. **编辑产品**
   - 后台：`/admin/product/update` → 直接更新主表
   - C端验证：刷新后显示最新数据

3. **下架产品**
   - 后台：`/product/offline/{id}` → 设置 `disabledFlag = true`
   - C端验证：立即从 `/productos` 列表中消失，直接访问 URL 返回 404

4. **上架产品**
   - 后台：`/product/online/{id}` → 设置 `disabledFlag = false`
   - C端验证：立即在 `/productos` 列表中显示

5. **删除产品**
   - 后台：`/admin/product/delete/{id}` → 物理删除主表记录
   - C端验证：立即从所有页面消失

6. **审批产品草稿**
   - 后台：`/product/draft/approve/{id}` → 写入主表 + 删除草稿
   - C端验证：立即在 `/productos` 列表中显示

7. **拒绝产品草稿**
   - 后台：`/product/draft/reject/{id}` → 草稿状态改为 REJECTED
   - C端验证：不显示在 `/productos` 列表中

#### 店铺维度

（流程与产品类似，关键字段：storeName, info, contacts, pictures, isCertified, vipLevel）

#### 农艺师维度

（流程与产品类似，关键字段：name, title, organization, profile, pictures, isCertified, vipLevel）

#### 品牌维度

（流程与产品类似，关键字段：brandName, brandCode, description, logoUrl, isCertified, vipLevel）

**注意：品牌无上下架功能**

### 3.2 COMPANY 操作流程

#### 前置条件：BASE → COMPANY 升级

1. **BASE 用户提交品牌申请**
   - 后台：`/brand-apply/create` → 创建品牌申请记录（`status=0` 待审核）
   - 上传营业执照等资料

2. **ADMIN 审批通过**
   - 后台：`/admin/brand-apply/approve/{userId}` → 授予 COMPANY 角色 + 绑定品牌 ID
   - 用户角色从 BASE 升级为 COMPANY

3. **COMPANY 用户同步产品**
   - 后台：`/product/sync-creator` → 把关联品牌的所有产品归属到自己
   - 原 BASE 用户失去这些产品的 CRUD 权限

#### 产品维度

1. **创建产品**
   - 后台：`/product/draft/create` → 创建草稿（`draftStatus=0`）
   - 自动标记 `isCertified=true`
   - 提交审核：`/product/draft/submit/{id}` → `draftStatus=1`
   - ADMIN 审批通过后写入主表
   - C端验证：审批通过后立即在 `/productos` 列表中显示

2. **编辑产品**
   - 后台：点击已发布产品 → 进入详情页 → 修改 → 保存草稿
   - 创建草稿（复制主表数据 + 用户修改）
   - 提交审核 → ADMIN 审批通过后更新主表
   - C端验证：审批通过后显示最新数据

3. **下架/上架产品**
   - 后台：`/product/offline/{id}` / `/product/online/{id}`
   - C端验证：立即生效

#### 店铺/农艺师维度（免审直发）

1. **创建店铺/农艺师**
   - 后台：`/store/draft/create` / `/agronomist/draft/create` → 创建草稿
   - 提交发布：`/store/draft/submit/{id}` → 免审直发（除非触发违禁词）
   - C端验证：立即在 `/tiendas` / `/agronomists` 列表中显示

2. **编辑店铺/农艺师**
   - 后台：修改 → 提交发布 → 免审直发
   - C端验证：立即显示最新数据

#### 品牌维度

1. **创建品牌**
   - 后台：`/brand/draft/create` → 创建草稿
   - 提交审核：`/brand/draft/submit/{id}` → 免审直发（除非触发违禁词）
   - C端验证：立即在 `/empresas` 列表中显示

#### 订阅冻结场景

1. **订阅到期**
   - ADMIN 设置 `subscription_expired_at < NOW()`
   - COMPANY 用户尝试创建草稿 → 返回错误提示
   - 已发布内容仍然可见，但无法编辑

### 3.3 BASE 操作流程

#### 产品维度

1. **创建产品**
   - 后台：`/product/draft/create` → 创建草稿（`draftStatus=0`）
   - 默认 `isCertified=false`
   - 提交审核：`/product/draft/submit/{id}` → `draftStatus=1`
   - ADMIN 审批通过后写入主表
   - C端验证：审批通过后立即在 `/productos` 列表中显示

2. **编辑产品**
   - 后台：点击已发布产品 → 进入详情页 → 修改 → 保存草稿
   - 提交审核 → ADMIN 审批通过后更新主表
   - C端验证：审批通过后显示最新数据

3. **下架/上架产品**
   - 后台：`/product/offline/{id}` / `/product/online/{id}`
   - C端验证：立即生效

#### 店铺/农艺师维度（免审直发）

（流程与 COMPANY 类似，但 `isCertified=false`）

---

## 四、C端关键字段验证清单

### 4.1 产品详情页（/productos/[slug]）

**必须验证的字段：**

1. **基础信息**
   - `productName` - 产品名称（H1 标题）
   - `sku` - SKU 编号
   - `brandName` - 品牌名称（Badge，可点击跳转）
   - `brandLogoUrl` - 品牌 Logo（显示在 Badge 中）
   - `pictures` - 产品图片（主图 + 多图）
   - `info` - 简述（纯文本）
   - `description` - 详细描述（富文本 HTML）

2. **价格与库存**
   - `minPrice` - 最低价格
   - `maxPrice` - 最高价格
   - `stockQuantity` - 库存数量（显示"有货"/"缺货"）

3. **化学成分**
   - `compositions` - 化学成分列表（可点击跳转到产品列表）

4. **标签**
   - `tags` - 标签列表（按 categoryName 分组，可点击跳转）

5. **认证与 VIP**
   - `isCertified` - 认证标识（显示认证徽章）
   - `vipLevel` - VIP 等级（影响搜索排序）

6. **时间戳**
   - `updateTime` - 更新时间（显示在页面底部）

7. **关联数据**
   - 产品用途（`ProductUsageVO`）
   - 产品文档（`ProductDocumentVO`）
   - 推荐农艺师（基于产品关联的农作物/虫害）
   - Where to Buy（VIP 店铺列表）

### 4.2 店铺详情页（/tiendas/[slug]）

**必须验证的字段：**

1. **基础信息**
   - `storeName` - 店铺名称
   - `info` - 店铺简介（富文本 HTML）
   - `pictures` - 店铺图片
   - `contacts` - 联系方式（姓名、电话、邮箱）

2. **认证与 VIP**
   - `isCertified` - 认证标识
   - `vipLevel` - VIP 等级（影响是否出现在产品详情页）

3. **地址信息**
   - `address` - 地址（如果有）

### 4.3 农艺师详情页（/agronomists/[slug]）

**必须验证的字段：**

1. **基础信息**
   - `name` - 姓名
   - `title` - 职称
   - `organization` - 所属机构
   - `profile` - 个人简介（富文本 HTML）
   - `pictures` - 头像/照片

2. **认证与 VIP**
   - `isCertified` - 认证标识
   - `vipLevel` - VIP 等级（影响是否出现在推荐列表）

### 4.4 品牌详情页（/empresas/[slug]）

**必须验证的字段：**

1. **基础信息**
   - `brandName` - 品牌名称
   - `brandCode` - 品牌代码
   - `description` - 品牌描述
   - `logoUrl` - 品牌 Logo

2. **认证与 VIP**
   - `isCertified` - 认证标识
   - `vipLevel` - VIP 等级

---

## 五、SEO 验证清单

### 5.1 页面元数据

**每个详情页必须验证：**

1. **`<title>` 标签**
   - 格式：`{内容名称} | Casalyin`
   - 不能硬编码，必须动态生成

2. **`<meta name="description">` 标签**
   - 内容：从 `info` 或 `description` 字段提取前 160 字符
   - 不能为空

3. **Open Graph 标签**
   - `og:title` - 内容名称
   - `og:description` - 简述
   - `og:image` - 主图 URL
   - `og:url` - 当前页面 URL

4. **结构化数据（JSON-LD）**
   - 产品：`Product` schema
   - 店铺：`LocalBusiness` schema
   - 农艺师：`Person` schema
   - 品牌：`Organization` schema

### 5.2 URL 结构

**必须验证：**

1. **URL 格式**
   - 产品：`/productos/{slug}?id={productId}`
   - 店铺：`/tiendas/{slug}?id={storeId}`
   - 农艺师：`/agronomists/{slug}?id={agronomistId}`
   - 品牌：`/empresas/{slug}?id={brandId}`

2. **Slug 生成规则**
   - 使用 `titleToSlug()` 函数
   - 小写、连字符分隔、移除特殊字符

3. **Canonical URL**
   - 每个页面必须有 `<link rel="canonical">` 标签

### 5.3 图片优化

**必须验证：**

1. **`alt` 属性**
   - 所有图片必须有 `alt` 属性
   - 内容：`{内容名称}` 或 `{内容名称} {序号}`

2. **图片加载**
   - 使用 Next.js `<Image>` 组件（自动优化）
   - 或使用 `loading="lazy"` 属性

3. **图片尺寸**
   - 主图：建议 1200x900（4:3）
   - 缩略图：建议 300x300（1:1）

### 5.4 加载速度

**必须验证：**

1. **首屏加载时间（LCP）**
   - 目标：< 2.5 秒
   - 使用 Chrome DevTools Lighthouse 测试

2. **服务端渲染（SSR）**
   - 产品/店铺/农艺师/品牌详情页必须使用 SSR
   - 不能全部客户端渲染

3. **API 响应时间**
   - 产品详情 API：< 500ms
   - 列表 API：< 1000ms

---

## 六、测试脚本结构设计

### 6.1 目录结构

```
casalyin-server/e2e/master-suite/
├── setup/
│   ├── create-test-accounts.spec.ts      # 创建测试账号（BASE + COMPANY）
│   └── cleanup.spec.ts                   # 清理测试数据
├── admin/
│   ├── product-crud.spec.ts              # ADMIN 产品 CRUD
│   ├── store-crud.spec.ts                # ADMIN 店铺 CRUD
│   ├── agronomist-crud.spec.ts           # ADMIN 农艺师 CRUD
│   ├── brand-crud.spec.ts                # ADMIN 品牌 CRUD
│   └── draft-approval.spec.ts            # ADMIN 草稿审批
├── company/
│   ├── product-workflow.spec.ts          # COMPANY 产品完整流程
│   ├── store-workflow.spec.ts            # COMPANY 店铺完整流程
│   ├── agronomist-workflow.spec.ts       # COMPANY 农艺师完整流程
│   ├── brand-workflow.spec.ts            # COMPANY 品牌完整流程
│   └── subscription-freeze.spec.ts       # COMPANY 订阅冻结场景
├── base/
│   ├── product-workflow.spec.ts          # BASE 产品完整流程
│   ├── store-workflow.spec.ts            # BASE 店铺完整流程
│   └── agronomist-workflow.spec.ts       # BASE 农艺师完整流程
├── frontend-validation/
│   ├── product-detail-fields.spec.ts     # 产品详情页字段验证
│   ├── store-detail-fields.spec.ts       # 店铺详情页字段验证
│   ├── agronomist-detail-fields.spec.ts  # 农艺师详情页字段验证
│   ├── brand-detail-fields.spec.ts       # 品牌详情页字段验证
│   └── seo-validation.spec.ts            # SEO 验证（meta 标签、结构化数据）
└── integration/
    ├── online-offline-sync.spec.ts       # 上下架同步验证
    ├── vip-display.spec.ts               # VIP 展示验证
    └── certified-badge.spec.ts           # 认证标识验证
```

### 6.2 测试执行顺序

1. **setup/create-test-accounts.spec.ts** - 创建测试账号
2. **admin/** - ADMIN 操作测试（优先级最高）
3. **company/** - COMPANY 操作测试
4. **base/** - BASE 操作测试
5. **frontend-validation/** - C端字段验证
6. **integration/** - 集成测试
7. **setup/cleanup.spec.ts** - 清理测试数据

### 6.3 测试数据管理

**原则：每个测试脚本独立创建和清理数据**

1. **测试账号**
   - BASE: `test-base-{timestamp}@test.com`
   - COMPANY: `test-company-{timestamp}@test.com`
   - ADMIN: `custom` / `123123`（固定）

2. **测试数据命名规范**
   - 产品：`[TEST] Product {timestamp}`
   - 店铺：`[TEST] Store {timestamp}`
   - 农艺师：`[TEST] Agronomist {timestamp}`
   - 品牌：`[TEST] Brand {timestamp}`

3. **清理策略**
   - 每个测试脚本结束后清理自己创建的数据
   - 使用 `test.afterAll()` 钩子
   - 如果测试失败，仍然执行清理

---

## 七、关键问题与待确认事项

### 7.1 已确认

✅ 测试账号：由测试脚本自动创建（BASE + COMPANY）
✅ C端验证重点：字段正确 + SEO + 加载速度
✅ 测试环境：仅本地（localhost）
✅ 业务规则：草稿状态机、上下架逻辑、VIP 和认证机制

### 7.2 待确认

❓ **违禁词库**：是否需要在测试中验证违禁词检测？
❓ **通知系统**：ADMIN 下架后是否需要验证用户收到通知？
❓ **性能指标**：LCP < 2.5s 是否是硬性要求？
❓ **图片 CDN**：测试环境是否使用阿里云 OSS？

---

## 八、下一步行动

1. **用户确认本报告** - 确认业务流程梳理是否准确
2. **编写测试脚本** - 按照上述结构编写 Playwright 测试
3. **执行测试** - 运行测试并记录问题
4. **自动修复** - 对于不符合业务的小问题，自动修复
5. **生成测试报告** - 汇总测试结果和问题清单
6. **建立测试规范文档** - 形成可定期执行的测试规范

---

**报告生成完毕，等待用户确认。**
