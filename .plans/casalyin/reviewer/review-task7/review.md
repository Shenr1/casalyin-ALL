# 代码审查：Task #7 — 品牌认领后批量同步产品归属

> 审查人: reviewer
> 日期: 2026-04-12
> 目标: 3 处后端变更（权限码修复 + brandId 回写 + 品牌查找增强）

## 变更清单

| # | 文件 | 变更类型 |
|---|------|----------|
| 1 | ProductController.java:298 | 权限码 `product:update` → `product:edit` |
| 2 | BrandApplyService.java:218-219 | 新建路径 `apply.setBrandId(brandId)` 回写 |
| 3 | ProductService.java:973-980 | 品牌查找逻辑：brandId 优先 + 品牌名 fallback |

---

## 审查结论：[OK] 通过

无 CRITICAL 或 HIGH 问题。

---

## 详细审查

### 修改 1：ProductController — 权限码修复

**变更**：`@SaCheckPermission("product:update")` → `@SaCheckPermission("product:edit")`（第 298 行，`/product/sync-creator` 接口）

**验证**：
- PermissionService.java 中 COMPANY 角色权限码列表包含 `product:edit`（第 47 行），不包含 `product:update`
- `product:update` 仅存在于 ADMIN 的兜底权限码（RoleConstant.java:48）
- 该接口语义为「COMPANY 角色将品牌产品归属到自己账号」，COMPANY 必须能调用
- 修复正确，权限码与角色匹配

**评定：✅ 正确**

### 修改 2：BrandApplyService — brandId 回写

**变更**：在 `approve()` 方法的新建品牌路径（else 分支，第 201-220 行），`getOrCreateBrandByName` 后添加 `apply.setBrandId(brandId)`

**验证**：
- 位置正确：在 `brandApplyDao.updateById(apply)` 之前（第 229 行），会随状态更新一并持久化
- 认领路径（if 分支，第 183-200 行）中 `apply.getBrandId()` 本身就不为 null，无需回写
- 新建路径之前缺少回写，导致 `brandId` 为 null，后续 `syncCreatorToBrandUser` 只能靠品牌名 fallback
- 修复后两条路径均保证 apply 表有 brandId，数据一致性提升

**评定：✅ 正确**

### 修改 3：ProductService — 品牌查找增强

**变更**：`syncCreatorToBrandUser` 方法中品牌查找逻辑从单一品牌名匹配改为 brandId 优先 + 品牌名 fallback

**验证**：
- brandId 查找使用 `brandDao.selectById(apply.getBrandId())`，主键查找，高效且唯一
- 仅当 brandId 为 null 或查不到记录时才 fallback 到品牌名匹配（兼容历史数据）
- 品牌名 fallback 保留了向后兼容性（未回写 brandId 的老数据仍可工作）
- 空值保护正确：先判断 `apply.getBrandId() != null` 再查询

**评定：✅ 正确**

---

## 项目规范检查

| 检查项 | 结果 |
|--------|------|
| 接口路径 kebab-case | ✅ `/product/sync-creator` 符合 |
| POST 用于写操作 | ✅ sync-creator 使用 POST |
| 动态参数用 `{id}` | ⚠️ 见下方 MEDIUM 问题（非本次变更引入） |
| 公开接口 @NoNeedLogin | ✅ 不涉及公开接口 |
| DB 变更需 Flyway | ✅ 无 DB 变更，不需要 Flyway |

---

## 发现问题

### MEDIUM-1：ProductController 路径参数命名不一致（既有问题，非本次变更引入）

**位置**：ProductController.java 第 248/259/280/304/313 行
**描述**：多个接口使用 `{productId}` 而非规范要求的 `{id}`，如 `/product/detail/{productId}`、`/product/offline/{productId}`
**规范**：`docs/backend/api-conventions.md` — 动态参数统一用 `{id}`（禁止 `{storeId}` 等业务前缀）
**建议**：后续统一修改，本次不阻断

### MEDIUM-2：ProductController `detailProduct` 方法 @PathVariable 参数名与路径参数不一致

**位置**：ProductController.java 第 248-249 行
**描述**：`@GetMapping("/product/detail/{productId}")` 方法参数名为 `@PathVariable Long productId`，虽然 Spring 能正确绑定，但与项目规范中统一用 `{id}` 的要求不符
**建议**：后续修改路径和参数名时一起处理

---

## 总结

三处变更逻辑正确、影响范围小、无数据安全风险。权限码修复是关键的 Bug Fix，确保 COMPANY 角色能正常调用同步产品接口。brandId 回写和查找增强形成闭环，提升了品牌认领→产品同步链路的鲁棒性。

**审查结果：[OK] 通过** — 可合并。
