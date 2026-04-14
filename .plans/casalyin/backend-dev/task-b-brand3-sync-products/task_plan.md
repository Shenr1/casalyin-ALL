# B-品牌3 后端：品牌认领后批量同步产品归属

> 状态: ✅ 已完成
> 日期: 2026-04-12

## 任务描述

COMPANY 用户认领品牌后，可触发"同步产品"：把所有关联该 brandId 的产品的 userId 更新为自己。

## 检查结论

功能已存在（接口 + Service + DAO + Mapper SQL 齐全），但发现两个 Bug 需修复。

### 已有代码

| 层 | 位置 | 功能 |
|----|------|------|
| Controller | `POST /product/sync-creator` | COMPANY 用户手动触发 |
| Service | `ProductService.syncCreatorToBrandUser()` | 查 apply → 找 brand → 批量更新产品 ownerUserId |
| DAO | `ProductDao.updateOwnerUserIdByBrandId()` | 批量 SQL |
| Mapper | `ProductMapper.xml` 第 613 行 | `UPDATE t_product SET owner_user_id = ? WHERE brand_id = ?` |

### 修复的问题

#### Bug 1：权限码错误（COMPANY 用户无法调用）

**文件**：`ProductController.java`

**原因**：`@SaCheckPermission("product:update")` — COMPANY 角色没有 `product:update` 权限码（只有 `product:edit`），导致 COMPANY 用户调用此接口会被权限拦截。

**修复**：改为 `@SaCheckPermission("product:edit")`

#### Bug 2：approve 新建路径未回写 brandId

**文件**：`BrandApplyService.java`

**原因**：approve 流程中，新建品牌路径（apply.brandId 为 null）通过 `getOrCreateBrandByName` 创建品牌后，没有把 brandId 回写到 apply 表。导致后续 syncCreatorToBrandUser 无法通过 apply.brandId 找到品牌，只能 fallback 用名称匹配（不够可靠）。

**修复**：在 approve 新建路径中增加 `apply.setBrandId(brandId)`，在后续 `brandApplyDao.updateById(apply)` 时会一并持久化。

#### 增强 3：品牌查找双重保障

**文件**：`ProductService.java`

**原因**：原逻辑只用品牌名匹配，如果品牌改名则失败。

**修复**：改为优先用 `apply.getBrandId()` 查找品牌，fallback 用品牌名匹配。

## 变更文件摘要

| 文件 | 变更 |
|------|------|
| ProductController.java | 权限码 `product:update` → `product:edit` |
| ProductService.java | 品牌查找逻辑增强：brandId 优先 + 名称 fallback |
| BrandApplyService.java | approve 新建路径回写 brandId 到 apply 表 |

## 无 DB 变更

t_brand_apply 表已有 brand_id 字段，t_product 表已有 owner_user_id 字段。无需 Flyway 文件。
