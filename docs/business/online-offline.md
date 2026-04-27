# 上下架规范

> 来源：business_online_offline.md + online-offline-unification-plan.md
> 最后更新：2026-04-24

---

## 业务语义

- **下架 = 移入草稿表**：下架的产品会从主表（`t_product`）移到草稿表（`t_product_draft`），不再展示
- **上架 = 发布到主表**：草稿审核通过后写入主表（`t_product`），`status = 1`（上架）
- 品牌维度：**无上下架功能**（品牌是用户身份标识，不是内容）

### 数据存储规则

| 状态 | 存储位置 | 表字段 | 说明 |
|------|---------|--------|------|
| 已发布（上架） | 主表 `t_product` | `status = 1` | 已发布且上架的产品 |
| 未发布（草稿） | 草稿表 `t_product_draft` | `draftStatus = 0` | 草稿，已保存未提交 |
| 未发布（待审核） | 草稿表 `t_product_draft` | `draftStatus = 1` | 待审核，已提交等待 admin 操作 |
| 未发布（已拒绝） | 草稿表 `t_product_draft` | `draftStatus = 2` | 已拒绝，可继续修改再提交 |

**关键规则：**
- 主表只保留上架的产品（`status = 1`）
- 下架的产品会移到草稿表，不保留在主表
- 列表页"已发布"tab 显示主表数据，"未发布"tab 显示草稿表数据

---

## 接口规范（目标态）

四个维度统一，**无 Admin 专属接口**：

```
POST /{dimension}/offline/{id}   RequestBody(required=false): { reason?: string }
POST /{dimension}/online/{id}    无 body
```

前端 API 函数命名规范：
```typescript
apiOnlineProduct(id)            apiOfflineProduct(id, reason?)
apiOnlineStore(id)              apiOfflineStore(id, reason?)
apiOnlineAgronomist(id)         apiOfflineAgronomist(id, reason?)
```

**已废弃，不再使用：**
- `apiAdminOfflineProduct`
- `apiAdminOfflineStore`
- `apiOfflineMyAgronomist` / `apiOnlineMyAgronomist`
- `apiAdminToggleAgronomistDisabled`

---

## 前端交互规范

| 操作 | 确认弹窗 | 理由输入框 | 成功后动作 |
|------|---------|-----------|-----------| 
| 用户上架 | ✅ 有 | ❌ 无 | 重新加载数据 |
| 用户下架 | ✅ 有 | ❌ 无 | 重新加载数据 |
| Admin 上架 | ✅ 有 | ❌ 无 | 重新加载数据 |
| Admin 下架 | ✅ 有 | ✅ 必填 | 重新加载数据 |

Admin 下架后后端自动发 `CONTENT_OFFLINE` 通知给归属用户。
通知内容：「您的[内容名称]已被管理员下架，理由：[reason]」

---

## 后端行为规范

- 接口收到请求后，通过 `SmartRequestUtil.getRequestUser()` 判断当前操作者角色
- 如果是 Admin：执行下架 + 发通知（`NotificationService.sendOfflineNotice()`）
- 如果是普通用户：只执行下架，不发通知
- 所有维度下架统一为：`disabledFlag = true`，**不退回草稿表、不删主表记录**

---

## 每个 Editor 页面的 handler 模式

每个详情页只需两个 handler，Admin/用户共用：

```typescript
// handleOnline：Admin 和用户共用，都有确认弹窗，无理由
// handleOffline：Admin 必填理由（Input.TextArea），用户无理由，共用同一 handler

// ContentDetailLayout props 写法：
onOnline={isOnline ? undefined : handleOnline}
onOffline={isOnline ? handleOffline : undefined}
```

---

## 改造进度

| 期 | 内容 | 状态 |
|----|------|------|
| 第一期 | 后端：统一接口、加 reason body、Admin 发通知、删 Admin 专属接口 | ✅ 已完成 |
| 第二期 | 前端 API 层重命名 + AgronomistDetail.tsx 运行时 bug 修复 | ⏳ 待开发 |
| 第三期 | 前端页面交互统一（ProductEditor / StoreEditor / AgronomistEditor） | ⏳ 待开发 |

> AgronomistDetail.tsx 已知 bug：`handleToggleDisabled` 等 4 个函数被引用但未定义，会产生运行时错误。
