# Task B2: isCompanyUser() 抽离为 RoleUtils 工具类

## 目标
将后端多处重复的 `isCompanyUser()` 和 `isAdminOrCompany()` 角色判断逻辑，统一抽离到 `RoleUtils` 静态工具类。

## 状态：已完成

## 变更清单

### 新增文件
- `com.casalyin.admin.module.system.role.constant.RoleUtils`
  - `isCompanyUser(RequestUser)` — instanceof 检查 + getRoleTypes().contains(COMPANY)
  - `isAdminOrCompany(RequestUser)` — ADMIN UserType 或 COMPANY role

### 重构文件（7 个）

| 文件 | 变更 |
|------|------|
| `ProductService.java` | 替换 3 处调用，删除 `isCompanyUser` + `isAdminOrCompany` 私有方法 |
| `ProductDraftService.java` | 替换 3 处调用，删除 `isCompanyUser` + `isAdminOrCompany` 私有方法 |
| `StoreDraftService.java` | 替换 3 处调用，删除 `isCompanyUser` 私有方法 |
| `StoreService.java` | 替换 1 处 inline instanceof+getRoleTypes 逻辑为 `RoleUtils.isCompanyUser()` |
| `BrandDraftService.java` | 替换 1 处调用，删除 `isCompanyUser` 私有方法 |
| `AgronomistDraftService.java` | 替换 2 处调用，删除 `isCompanyUser` 私有方法 |
| `BrandApplyService.java` | 替换 1 处 inline getRoleTypes 逻辑为 `RoleUtils.isCompanyUser()` |

### 验证
- grep 确认 business 目录下无残留 `private.*isCompanyUser` / `private.*isAdminOrCompany` 方法
- grep 确认无残留 `getRoleTypes().contains(RoleTypeEnum.COMPANY)` inline 检查
- 所有 15 处调用统一引用 RoleUtils

## 备注
纯重构，无业务逻辑变更，无 DB 变更，不需要 reviewer 审查。
