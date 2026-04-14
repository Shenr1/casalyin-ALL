# 数据库规范

> 来源：db_migration.md
> 最后更新：2026-04-11

---

## Flyway 迁移规范

### 现状

Flyway 已于 2026-04-06 接入，基线为 V1（来自 DMS 导出的真实数据库结构）。

**Why:** 之前散落的 migration_*.sql 文件无版本顺序，手动执行容易遗漏，导致 Entity 与数据库不同步。

### 迁移文件位置

```
casalyin-java/casalyin-admin/src/main/resources/db/migration/
  V1__baseline.sql   ← 基线，禁止修改
  V2__xxx.sql        ← 往后新增
  V3__xxx.sql
  ...
```

### 迁移规则

1. 有任何数据库变更 → **只写 `V{n}__描述.sql` 文件**，不手动在 DMS / 数据库客户端执行 SQL
2. 版本号必须递增，不能跳也不能重复
3. 已提交的文件不能修改（Flyway 校验 checksum，改了启动报错）
4. 写错了不能改旧文件，要新建下一个版本来修正
5. 命名语义化，如 `V2__add_product_creator_id.sql`

### 配置位置

- 依赖：`casalyin-admin/pom.xml` → `flyway-mysql`
- 配置：`sa-base/src/main/resources/dev/sa-base.yaml` → `spring.flyway`
- `baseline-on-migrate: true`（存量数据库首次打基线，不重跑 V1）
- `validate-on-migrate: true`（每次启动校验已执行文件的 checksum）

### 旧文件

`data/migration_*.sql` 那批文件保留作历史参考，不再使用。

---

## 核心表结构速查

### 业务状态字段约定

| 字段名 | 类型 | 含义 | 适用表 |
|--------|------|------|--------|
| `disabled_flag` | boolean | 是否下架（true=已下架） | t_product、t_store、t_agronomist |
| `is_certified` | boolean | 是否认证 | t_product、t_store、t_agronomist、t_brand |
| `draft_status` | int | 草稿状态（0草稿/1待审核/2已拒绝） | 四个维度的 `_draft` 表 |
| `reject_reason` | varchar | 审核拒绝原因 | 四个维度的 `_draft` 表 |
| `vip_level_id` | bigint | VIP 等级关联 | t_vip_subscription |

### 草稿表命名规则

四个维度的草稿表：

```
t_product_draft      ← 产品草稿
t_store_draft        ← 店铺草稿
t_agronomist_draft   ← 农艺师草稿
t_brand_draft        ← 品牌草稿
```

### COMPANY 订阅表

```
t_brand_apply        ← 品牌申请（B→C升级申请 + 订阅信息）
  subscription_status         ← 0=未订阅 / 1=已订阅 / 2=已过期
  subscription_expired_at     ← 订阅到期时间
```

---

## 字段命名规范

- 字段名全部 **snake_case**
- 布尔类型用 `is_` / `has_` 前缀（如 `is_certified`、`disabled_flag`）
- 时间字段用 `_at` 后缀（如 `created_at`、`expired_at`）
- 外键字段用 `_id` 后缀（如 `user_id`、`brand_id`、`vip_level_id`）
- Entity 字段用 **camelCase**（MyBatis Plus 自动映射）

---

## 注意事项

- **禁止手动在生产或测试库执行 SQL**，所有变更必须通过 Flyway 迁移文件
- 本地数据库：`casalyin_admin_v1`
- MyBatis Plus 逻辑删除字段：确认表是否启用（避免物理删除和逻辑删除混用）
