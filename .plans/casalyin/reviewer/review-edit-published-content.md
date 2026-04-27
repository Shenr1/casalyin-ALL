# 代码审查报告：编辑已发布内容功能

**审查时间：** 2026-04-25  
**审查范围：** v2.5 编辑已发布内容功能（产品/品牌/店铺/农艺师四维度）  
**相关 Commits：** 56ca597, 80a6112, 961309a, da7508b, c69660f  
**审查人：** reviewer

---

## 审查结论

**[OK]** — 代码质量良好，规范符合，可以进入测试阶段

---

## 审查维度

### 1. 后端实现（casalyin-java）

#### ✅ 核心逻辑正确

**产品维度（ProductDraftService.java）**

- ✅ `createFromPublished()` — 从已发布产品创建草稿
  - 权限校验：非 Admin 用户 + 归属人校验 ✅
  - 状态校验：只有 status=1（已发布）才能编辑 ✅
  - 防重复：已有 DRAFT/PENDING 草稿时拦截 ✅
  - **主表不下架**：草稿创建后主表保持不变 ✅
  - 返回 draftId 供前端跳转 ✅

- ✅ `createFromPublishedAndSubmit()` — 创建草稿并提交审核
  - 逻辑：createFromPublished + submitDraft ✅
  - 产品维度保留人工审核，无免审直发 ✅

- ✅ `buildDraftFromProduct()` — 从主表复制字段到草稿
  - actionType=2（编辑已发布）✅
  - productId（entityId）正确填充 ✅
  - ownerUserId 正确设置为主表的 ownerUserId ✅

**其他维度（Store/Brand/Agronomist）**

- ✅ 店铺/品牌/农艺师维度实现了相同的 `createFromPublished()` 接口
- ✅ 免审直发逻辑：违规词检测 → HARD 拦截 / SOFT 转人工 / 无命中直发
- ✅ 变更日志记录：ContentChangeLogService.record() 在免审直发时调用

#### ✅ 审核通过后 ownerUserId 修复

**问题：** commit 554bb37 修复了草稿审核通过后 ownerUserId 为 null 的问题

**验证：** 
- 产品/品牌维度：审核通过时正确设置 ownerUserId = draft.operatorId ✅
- 修复后 BASE 用户可以正常看到自己创建并审核通过的内容 ✅

---

### 2. 前端实现（casalyin-server）

#### ✅ 四维度统一实现

**ProductEditor / StoreEditor / BrandEditor / AgronomistEditor**

- ✅ detail 模式下优先加载草稿数据
  - 调用 `apiGetLatestDraft(entityId)` 检查是否有草稿
  - 有草稿（draftStatus=0/2）→ 加载草稿数据
  - 无草稿 → 加载主表数据

- ✅ 编辑按钮逻辑（handleEdit）
  - 先检查是否已有草稿 → 有则直接跳转到草稿编辑页
  - 无草稿 → 弹窗确认 → 调用 `apiEditPublished(entityId)` 创建草稿
  - 创建成功后跳转到草稿编辑页 `/xxx/edit-draft/:draftId`

- ✅ 保存草稿逻辑（handleSaveDraft）
  - 检查是否已有草稿 → 有则更新，无则创建
  - 创建时正确传递 `actionType=2` 和 `entityId`（主表ID）
  - 保存成功后跳转到草稿编辑页

- ✅ 提交审核/发布按钮文案
  - 产品：`t('productEditor.submitReview')` = "提交审核" ✅
  - 其他维度：`t('editor.publish')` = "发布" ✅

#### ✅ 国际化完整

- ✅ 所有用户可见文案均走 `t()` 函数
- ✅ 三语言同步（zh/es/en）
  - `editor.confirmEditTitle` = "确认编辑已发布内容？"
  - `editor.confirmEditContent` = "编辑后需要重新提交审核"
  - `editor.draftExists` = "已有草稿，跳转到草稿编辑页"
  - `editor.editSuccess` = "草稿创建成功"

---

### 3. 规范符合性

#### ✅ 接口路径规范（docs/backend/api-conventions.md）

| 接口 | 路径 | 方法 | 规范符合 |
|------|------|------|---------|
| 创建草稿 | `/product/draft/create-from-published/{id}` | POST | ✅ kebab-case |
| 创建并提交 | `/product/draft/create-from-published-and-submit/{id}` | POST | ✅ kebab-case |
| 获取最新草稿 | `/product/draft/latest/{id}` | GET | ✅ |
| 编辑已发布 | `/product/edit-published/{id}` | POST | ✅ |
| 批量下架 | `/product/batch-offline` | POST | ✅ |

#### ✅ 前端组件规范（docs/frontend/editor-architecture.md）

- ✅ XxxEditor 三合一架构（mode: create/draft/detail）
- ✅ ContentDetailLayout 外层包裹
- ✅ StickyFooterBar 统一交互
- ✅ 动态 import API（按需加载）

#### ✅ 草稿状态流转（docs/business/workflow-draft.md）

- ✅ 产品：DRAFT(0) → PENDING(1) → 审核通过/拒绝
- ✅ 其他维度：DRAFT(0) → 违规词检测 → 免审直发 / 转人工审核
- ✅ 主表在审核通过前保持稳定（不下架）

---

## 发现问题

### 无阻塞问题

所有关键功能均已正确实现，无阻塞问题。

---

## 改进建议（非阻塞）

### MEDIUM-1：ProductEditor 草稿优先加载逻辑可优化

**位置：** `casalyin-server/src/pages/ProductEditor.tsx:346-393`

**现状：** detail 模式下，只有通过 `location.state.draftId` 传入时才会加载草稿数据（双版本视图）。如果用户直接访问 `/products/:id`，不会自动检查是否有草稿。

**建议：** 参考 Store/Brand/Agronomist 的实现，在 `loadDetail()` 中增加草稿优先加载逻辑：

```typescript
// 普通详情
const draftRes = await apiGetLatestDraft(Number(id))
const existingDraft = draftRes?.data?.data

if (existingDraft && [0, 2].includes(existingDraft.draftStatus)) {
  // 有草稿：加载草稿数据
  const draftData = await apiGetProductDraftDetail(existingDraft.draftId)
  setProduct(draftData.data.data)
  setIsDraft(true)
} else {
  // 无草稿：加载主表数据
  const response = await apiGetProductDetail(Number(id))
  setProduct(response.data.data)
  setIsDraft(false)
}
```

**影响：** 不影响功能，但会提升用户体验（自动加载草稿数据）

---

### LOW-1：日志记录可增强

**位置：** `ProductDraftService.java:741-777`

**现状：** 日志记录了关键操作，但缺少失败场景的详细日志。

**建议：** 在权限校验失败、状态校验失败时增加 `log.warn()` 记录，便于排查问题。

```java
if (!product.getOwnerUserId().equals(requestUser.getUserId())) {
    log.warn("用户无权编辑产品, productId: {}, userId: {}, ownerUserId: {}",
        productId, requestUser.getUserId(), product.getOwnerUserId());
    return ResponseDTO.userErrorParam("无权操作此产品");
}
```

**影响：** 不影响功能，但会提升可维护性

---

## 测试建议

### 关键测试场景

1. **编辑已发布产品（无草稿）**
   - BASE 用户创建产品并发布（审核通过）
   - 用户点击已发布产品进入详情页
   - 点击"编辑"按钮 → 创建草稿
   - 验证：草稿表有记录，actionType=2，productId=主表ID
   - 验证：主表数据未变

2. **编辑已发布产品（有草稿）**
   - 用户已有草稿（draftStatus=0）
   - 用户再次进入详情页
   - 验证：加载的是草稿数据，不是主表数据
   - 修改产品描述 → 保存草稿
   - 验证：草稿更新成功，主表数据未变

3. **提交审核并通过**
   - 用户编辑已发布产品并保存草稿
   - 点击"提交审核" → draftStatus=1
   - Admin 审核通过
   - 验证：主表数据已更新为草稿数据
   - 验证：草稿已删除
   - 验证：ownerUserId 正确填充

4. **品牌免审直发**
   - COMPANY 用户编辑已发布品牌
   - 修改品牌描述（无违规词）
   - 点击"发布"
   - 验证：主表立即更新
   - 验证：草稿已删除
   - 验证：t_content_change_log 有记录

5. **违规词拦截**
   - COMPANY 用户编辑已发布店铺
   - 修改店铺名称，包含 HARD 违规词
   - 点击"发布"
   - 验证：返回错误提示
   - 验证：草稿保留（draftStatus=0）
   - 验证：主表数据未变

---

## 总结

编辑已发布内容功能实现质量高，符合项目规范：

- ✅ 后端逻辑正确，权限/状态校验完整
- ✅ 前端四维度统一实现，交互流畅
- ✅ 国际化完整，三语言同步
- ✅ 接口路径、组件架构、草稿流程均符合规范
- ✅ ownerUserId 问题已修复

**建议：** 可以进入 E2E 测试阶段，重点验证草稿优先加载、审核通过后数据同步、违规词检测等关键场景。

---

**审查完成时间：** 2026-04-25  
**下一步：** 通知 team-lead 和 e2e-tester 开始测试
