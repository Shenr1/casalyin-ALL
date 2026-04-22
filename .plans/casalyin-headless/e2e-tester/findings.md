# e2e-tester - 发现索引

> 纯索引——每个条目应简短（Status + Report 链接 + Summary）。

---

<初始为空，工作中填写>

## test-full-business-flow (2026-04-22)
- Status: complete
- Pass rate: 19/19 (100%)
- Test file: casalyin-server/e2e/v2.3/full-business-flow.spec.ts
- Summary: 全链路通过。

### BUG-1 排查结果（店铺/农艺师 submit 响应捕获）

**结论：店铺/农艺师的 create + submit 流程正常工作，不是 Bug。**

用 `waitForResponse` 捕获到的实际响应：

**店铺 (A2)**:
```
store/draft/create → status=200, ok=true, code=0, data={"draftId":19}
store/draft/submit → status=200, ok=true, code=0, data="提交成功，已直接发布"
```

**农艺师 (A3)**:
```
agronomist/draft/create → status=200, ok=true, code=0, data=62
agronomist/draft/submit → status=200, ok=true, code=0, data="提交成功，已直接发布"
```

之前测试误判为"静默失败"的原因：`page.on('response')` 在 Vite proxy 下不可靠，未能捕获到响应。改用 `waitForResponse` 后确认两步都成功。

### 仍存在的真实 Bug

- **BUG-FIELD-MISMATCH**: 前端 `BatchSetVip.tsx` 发送 `userId` 但后端 `UserSubscriptionBatchAddForm` 期望 `ownerUserId`。前后端字段名不一致，后台 UI 批量设置 VIP 功能实际上是坏的。
  - 文件: `casalyin-server/src/pages/BatchSetVip.tsx:201` → `userId: selectedUser.userId`
  - 后端: `UserSubscriptionBatchAddForm.java:23` → `private Long ownerUserId`
  - 修复: 前端改为 `ownerUserId: selectedUser.userId`，或后端加 `@JsonAlias("userId")`

- **BUG-PRODUCT-DRAFT**: COMPANY 用户产品「提交审核」后，`/product/draft/my` 返回 0 条草稿。A1 测试中 UI 显示导航成功但未捕获到 draft create 的 API 响应。需进一步排查 ProductEditor 的 `handleSubmitAuditFromCreate` 流程。
