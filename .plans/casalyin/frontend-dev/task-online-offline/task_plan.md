# task-online-offline 任务计划

## 目标

上下架统一改造第二期（前端 API 层重命名）+ 第三期（三个页面交互统一）。

## 涉及文件（实际路径）

| 计划路径 | 实际路径 |
|---|---|
| src/api/product.ts | casalyin-server/src/api/product.ts |
| src/api/store.ts | casalyin-server/src/api/store.ts |
| src/api/agronomist.ts | casalyin-server/src/api/agronomist.ts |
| AgronomistDetail.tsx | 不存在，已合并为 AgronomistEditor.tsx（mode=detail/draft） |
| ProductEditor.tsx | casalyin-server/src/pages/ProductEditor.tsx |
| StoreDetail.tsx | 不存在，已合并为 StoreEditor.tsx（mode=detail） |
| AgronomistDraftEdit.tsx | 不存在，已合并为 AgronomistEditor.tsx（mode=draft） |

## 现状分析

### API 层
- product.ts：apiOfflineProduct(reason?) 已完成，apiAdminOfflineProduct 不存在 ✅
- store.ts：apiOfflineStore(reason?) 已完成，apiAdminOfflineStore 不存在（但 StoreEditor 代码仍 import 它！）
- agronomist.ts：apiOfflineAgronomist/apiOnlineAgronomist 已存在，但 apiOfflineMyAgronomist/apiOnlineMyAgronomist 标为 @deprecated 未删除

### AgronomistEditor.tsx（合并了 AgronomistDetail + AgronomistDraftEdit）
- handleOnline：无确认弹窗，用旧 API apiOnlineMyAgronomist，成功后 setIsOnline(true) 本地 patch ❌
- handleOffline：有确认弹窗，用旧 API apiOfflineMyAgronomist，成功后 setIsOnline(false) 本地 patch ❌，无 Admin 理由输入框 ❌

### ProductEditor.tsx
- handleProductOnline：有确认弹窗 ✅，调 apiOnlineProduct，成功后 loadDetail() ✅
- handleProductOffline：有确认弹窗 ✅，调 apiOfflineProduct（无 reason）❌，成功后 loadDetail() ✅
- onOnline/onOffline：Admin 无入口（!canAuditProduct 条件过滤掉了 Admin）❌

### StoreEditor.tsx
- handleUserOnline：无确认弹窗 ❌，成功后 setIsOffline(false) 本地 patch ❌
- handleUserOffline：有确认弹窗 ✅，apiOfflineStore 无 reason ❌，成功后 setIsOffline(true) 本地 patch ❌
- handleAdminOffline：有弹窗 ✅，必填理由 ✅，但调用 apiAdminOfflineStore（已不存在）❌，成功后 navigate('/stores') 跳列表 ❌

## 任务清单

- [x] 1. 删除 agronomist.ts 中 @deprecated 旧函数
- [x] 2. 修改 AgronomistEditor.tsx：handleOnline/handleOffline 规范化
- [x] 3. 修改 ProductEditor.tsx：handleProductOnline→handleOnline，Admin 入口打开
- [x] 4. 修改 StoreEditor.tsx：三个 handler 合并为两个，修复 apiAdminOfflineStore 引用
- [x] 5. 更新根 findings.md 索引

## 决策

- AgronomistEditor 既是 AgronomistDetail（mode=detail）也是 AgronomistDraftEdit（mode=draft），两种模式共用同一个 handleOnline/handleOffline，targetId 逻辑已有（`mode === 'detail' ? Number(paramId) : draft?.agronomistId`）
- StoreEditor 中 isOffline 本地 state 不删除（其他处也可能引用），但成功后改为调用 loadDetail() 而非 setIsOffline
