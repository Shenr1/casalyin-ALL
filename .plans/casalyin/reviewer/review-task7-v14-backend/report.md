# 代码审查报告 — Task #7 v1.4 期一（认证申请体系后端）

**日期**：2026-04-14  
**审查人**：reviewer  
**请求方**：team-lead（代 backend-dev）  
**涉及文件**：
- CertificationApplyController.java（新建）
- V8__add_certification_permissions.sql（新建）
- CertificationApplyForm.java（新增 resourceType 字段）
- CertificationApplyService.java（新增 cancelById()，apply() 签名调整）
- RoleConstant.java（ADMIN_DEFAULT_PERMISSION_CODES 追加）
- PermissionService.java（COMPANY/BASE PresetPermissions 追加）
- AdminSwaggerTagConst.java（新增常量）

---

## 最终结论

[WARN] — 无 CRITICAL / HIGH 问题；存在 2 处 MEDIUM + 1 处 LOW，不阻断合并，建议知悉。

---

## 通过项

### 接口路径规范 ✅
- 全部 kebab-case，动态参数统一用 `{id}`
- 上架/下架语义接口均用 POST（approve/reject/cancel 均 POST）
- 无 `@SaIgnore`、无 `@NoNeedLogin`（所有接口均需登录+权限码）

### 权限码三处同步 ✅
| 位置 | certification:apply/cancel/query | certification:approve/reject |
|------|----------------------------------|------------------------------|
| RoleConstant.ADMIN_DEFAULT_PERMISSION_CODES | ✅（L55，5个全部） | ✅ |
| PermissionService COMPANY_PERMISSION_CODES | ✅（L51-53） | —（Admin 专属，正确） |
| PermissionService BASE_PERMISSION_CODES | ✅（L69-71） | — （Admin 专属，正确） |
| PermissionService PresetPermissions | ✅（L269-273，5个全部） | ✅ |
| V8 SQL INSERT t_permission | ✅（5个全部） | ✅ |
| V8 SQL 角色赋权（BASE/COMPANY） | ✅（3个用户侧） | — （ADMIN 走兜底，正确） |

### cancelById() 逻辑 ✅
- 先查记录存在性 → 校验 applicantId == 当前用户 → 校验 status == PENDING → 删除
- 顺序正确，三层防护均已覆盖

### checkResourceOwner() switch-case 完整 ✅
- PRODUCT / STORE / AGRONOMIST / BRAND 四维度均已覆盖
- 校验资源存在性 + ownerUserId 归属，逻辑正确

### Flyway 迁移文件规范 ✅
- 文件名 V8__add_certification_permissions.sql 符合命名规范
- INSERT IGNORE 防重复写入 ✅
- CROSS JOIN + WHERE 子查询赋权风格与项目已有迁移文件一致 ✅
- 仅写入数据，无 DDL（表结构已在 V6 创建，注释已说明）✅

### @Transactional 覆盖 ✅
- apply()、cancelById()、approve()、reject() 均标注 `@Transactional(rollbackFor = Exception.class)`

---

## 问题清单

### MEDIUM-1：Service 中残留旧方法 cancel(resourceType, resourceId) 未删除

**文件**：`CertificationApplyService.java` L115-130

旧方法 `cancel(String resourceType, Long resourceId)` 已被 `cancelById(Long applyId)` 取代，Controller 只调用 `cancelById`，旧方法已无入口但仍存在于 Service 中。这是死代码，不影响运行，但造成混淆（两个语义相同的 cancel 方法，外部无法区分）。

**建议**：删除旧 `cancel(resourceType, resourceId)` 方法，或添加 `@Deprecated` 注解并注释标明。

---

### MEDIUM-2：getStatus() 接口鉴权不完整——非申请人的资源拥有者无法访问

**文件**：`CertificationApplyService.java` L160-177

当前鉴权逻辑：`latest.getApplicantId().equals(requestUser.getUserId()) || ADMIN`

由于 `applicantId` 在 `apply()` 中被设为当前调用用户（资源拥有者），对同一用户场景而言是正确的。但以下边缘场景存在问题：

> 若 Admin 曾代为发起申请（applicantId ≠ 资源实际拥有者 ownerUserId），则资源真实拥有者调用 `GET /certification/status` 时，会因 `applicantId != userId` 而被拒绝（返回"无权查看"），但该用户是资源的合法归属方。

虽然当前业务流程中 Admin 不会代发申请（申请需要 `@SaCheckPermission("certification:apply")`），理论上不会触发。但权限校验语义上应基于**资源归属**（ownerUserId）而非**申请人**（applicantId），以保证未来扩展时的正确性。

**建议**：getStatus() 中校验逻辑改为：当前用户是资源拥有者（需调用 checkResourceOwner 或查 ownerUserId 字段）或 ADMIN，而非基于 applicantId。

---

### LOW-1：list() 方法 N+1 查询

**文件**：`CertificationApplyService.java` L248-251

```java
for (CertificationApplyVO vo : voList) {
    vo.setResourceName(getResourceName(vo.getResourceType(), vo.getResourceId()));
}
```

每条记录单独查一次资源表（N 次 SQL），分页大时性能较差。当前分页场景较小，不阻断。

**建议**：后续可按 resourceType 分组批量查询，或在 DAO 层用 JOIN 一次取出。

---

## 审查完成
