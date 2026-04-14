# backend-dev - 工作日志

> 用于上下文恢复。压缩/重启后先读此文件。

---

## 2026-04-13 Session 3 — 品牌产品列表接口

### 完成的工作

新增 `GET /brand/my-products` 接口，供 COMPANY 品牌管理页展示产品卡片。

1. **新建 VO**：`BrandProductVO.java`（brand/domain/vo/）
   - 字段：productId、productName、coverUrl、vipLevelValue、disabledFlag、status

2. **ProductDao 新增方法**：`listByBrandIdTop12(@Param("brandId") Long brandId)`

3. **ProductMapper.xml 新增 SQL**：
   - `JSON_UNQUOTE(JSON_EXTRACT(p.pictures, '$[0]'))` 直接取封面图
   - 复用已有 VIP 联查逻辑（`t_user_subscription` + `t_vip_level`）
   - `ORDER BY COALESCE(vip.level_value, 0) DESC, p.create_time DESC LIMIT 12`

4. **BrandService 新增方法**：`getMyProducts()`
   - 从 token 取 userId → 查该用户绑定的品牌 → 查产品列表
   - 未绑定品牌时返回空列表

5. **BrandController 新增路由**：`GET /brand/my-products`
   - 无权限注解（只需登录，COMPANY 账号自然有品牌关联）

### 修改的文件清单
- `brand/domain/vo/BrandProductVO.java`（新建）
- `product/dao/ProductDao.java`（新增方法）
- `resources/mapper/ProductMapper.xml`（新增 SQL）
- `brand/service/BrandService.java`（新增 getMyProducts 方法）
- `brand/controller/BrandController.java`（新增路由）

### 无 DB 变更
- 无需 Flyway 迁移文件（所有涉及字段已存在）

---

## 2026-04-12 Session 2 — 品牌认证第一期后端实现

### 完成的工作

1. **Flyway 迁移 V4__add_brand_fields.sql**
   - `t_agronomist` 新增 `brand_id` 列
   - `t_agronomist_draft` 新增 `brand_id` 列
   - `t_product` 新增 `brand_owner_id` 列
   - 注：`t_store` 和 `t_store_draft` 已有 `brand_id` 列，无需修改

2. **实体类更新**
   - `AgronomistEntity` 新增 `brandId` 字段
   - `AgronomistDraftEntity` 新增 `brandId` 字段
   - `ProductEntity` 新增 `brandOwnerId` 字段

3. **COMPANY 创建内容自动写入 brand_id**
   - `StoreDraftService.createStoreDraft()` — COMPANY 用户保存草稿时自动查询并写入 brandId
   - `StoreDraftService.createNewDraft()` — 修改店铺时同理
   - `AgronomistDraftService.addDraft()` — COMPANY 用户创建农艺师草稿时自动写入 brandId

4. **COMPANY 提交草稿时跳过审核直接发布**
   - `StoreDraftService.submitDraft()` — COMPANY 用户提交即免审直发（即使命中 SOFT 违规词）
   - `AgronomistDraftService.submitDraft()` — 同上
   - `ProductDraftService.submitDraft()` — COMPANY 用户跳过 PENDING，直接调用 ProductAuditService 发布

5. **审核通过时同步 brandId**
   - `AgronomistAuditService.approve()` — 管理员审核通过时，brandId 从草稿同步到正式表
   - `AgronomistDraftService.directPublishAgronomist()` — 免审直发时同步 brandId
   - StoreAuditService 已有 brandId 同步逻辑，无需修改

### 修改的文件清单
- `casalyin-admin/src/main/resources/db/migration/V4__add_brand_fields.sql` （新建）
- `casalyin-admin/.../agronomist/domain/entity/AgronomistEntity.java`
- `casalyin-admin/.../agronomist/domain/entity/AgronomistDraftEntity.java`
- `casalyin-admin/.../product/domain/entity/ProductEntity.java`
- `casalyin-admin/.../store/service/StoreDraftService.java`
- `casalyin-admin/.../agronomist/service/AgronomistDraftService.java`
- `casalyin-admin/.../agronomist/service/AgronomistAuditService.java`
- `casalyin-admin/.../product/service/ProductDraftService.java`

### 待确认
- 由于环境中没有 JAVA_HOME，无法执行编译验证。需要用户重启后端确认编译通过。

---

## 2026-04-11 Session 1 — 初始化

- [x] 工作目录初始化完成
- [x] 等待 team-lead 分配任务
