# 已发布内容编辑功能规范

> 版本：v1.0  
> 创建日期：2026-04-23  
> 状态：待评审

---

## 一、业务背景

### 1.1 当前问题

用户想要修改已发布的内容时，流程不清晰：
- v2.4 实现了"保存到草稿"/"提交审核"两个按钮，但用户可能不理解这两个选项的区别
- "下架"操作只是设置 `disabledFlag=true`，不会创建草稿，导致下架后的内容无处可寻
- 状态概念混乱：已发布、已下架、草稿，用户难以理解

### 1.2 设计目标

**核心理念：简化状态，统一流程**

1. **状态简化**：只有两个核心状态（已发布、草稿），取消"下架"状态概念
2. **流程统一**：四个维度（产品/店铺/品牌/农艺师）遵循相同的编辑流程
3. **提前告知**：在用户编辑前明确告知需要重新审核，避免意外
4. **操作直观**：用户想编辑就点"编辑"，不需要理解"下架"的概念

---

## 二、核心概念

### 2.1 状态定义

**主状态（2个）：**
- **已发布（Published/Active）** - 内容在线可见，用户可以访问
- **草稿（Draft）** - 内容未发布或编辑中，用户不可见

**审核子状态（3个）：**
- **草稿（draftStatus=0）** - 可编辑
- **待审核（draftStatus=1）** - 不可编辑，等待审核
- **已拒绝（draftStatus=2）** - 可编辑，需要修改后重新提交

**取消的概念：**
- ❌ "下架"不再是一个状态，只是一个操作
- ❌ "已下架"的内容就是"草稿"状态

### 2.2 操作定义

**编辑已发布内容：**
```
点击"编辑"按钮
  ↓
自动下架（disabledFlag=true）
  ↓
创建草稿（从主表复制数据到草稿表）
  ↓
在当前页面进入编辑模式
```

**批量下架：**
```
选中多个内容 → 批量下架
  ↓
批量设置 disabledFlag=true
  ↓
批量创建草稿
  ↓
跳转到"草稿"列表
```

---

## 三、用户流程设计

### 3.1 流程 1：详情页编辑已发布内容

#### 步骤 1：初始状态（只读模式）

**页面元素：**
- 顶部提示条（Alert，info 类型）：
  ```
  此内容已发布。若需修改，请点击"编辑"按钮。
  注意：修改后需要重新提交审核才能发布。
  ```
- 右上角"编辑"按钮（替代原来的"保存"按钮位置）
- 所有表单字段为只读状态

#### 步骤 2：点击"编辑"按钮

**确认弹窗：**
- 标题：`确认编辑`
- 内容：
  ```
  编辑已发布内容需要重新提交审核。
  确认要编辑吗？
  ```
- 按钮：`[取消]` `[确认编辑]`

#### 步骤 3：确认后进入编辑模式

**后端操作：**
1. 主表设置 `disabledFlag=true`（下架）
2. 从主表复制数据到草稿表
3. 设置 `draftStatus=0`（草稿）
4. 设置 `actionType=2`（修改）
5. 返回 `draftId`

**前端操作：**
1. 调用后端接口获取 `draftId`
2. 在当前页面切换到编辑模式（不跳转）
3. 重新加载草稿数据
4. 顶部提示条变为警告类型（warning）：
   ```
   当前为草稿状态，需要提交审核后才能发布。
   ```
5. 底部操作栏显示：`[保存草稿]` `[提交审核]`

#### 步骤 4：用户编辑并保存

**保存草稿：**
- 保存到草稿表
- 停留在当前页面
- 提示："草稿已保存"

**提交审核：**
- 设置 `draftStatus=1`（待审核）
- 品牌/店铺/农艺师：如果是 COMPANY 用户且品牌已认证，自动审核通过并发布
- 产品：进入人工审核流程
- 跳转到"草稿"列表
- 提示："已提交审核"

### 3.2 流程 2：列表页批量下架

#### 步骤 1：选中内容

**页面元素：**
- 列表中的复选框
- 右上角操作下拉菜单：
  - 批量删除
  - 批量下架（新增）

#### 步骤 2：点击"批量下架"

**确认弹窗：**
- 标题：`确认批量下架`
- 内容：
  ```
  下架后将变为草稿状态，需要重新提交审核才能发布。
  确认下架 X 个内容吗？
  ```
- 按钮：`[取消]` `[确认下架]`

#### 步骤 3：批量下架成功

**后端操作：**
1. 批量设置 `disabledFlag=true`
2. 批量创建草稿记录

**前端操作：**
1. 跳转到"草稿"列表
2. 提示："X 个内容已下架并转为草稿"

### 3.3 流程 3：删除草稿并恢复上线

#### 场景说明

用户编辑已发布内容后创建了草稿，如果用户不想修改了，可以删除草稿。

#### 步骤 1：点击"删除草稿"按钮

**确认弹窗：**
- 标题：`确认删除草稿`
- 内容：
  ```
  删除草稿后，是否恢复内容上线？
  ```
- 按钮：`[仅删除草稿]` `[删除草稿并恢复上线]`

#### 步骤 2：用户选择

**选项 1：仅删除草稿**
- 删除草稿记录
- 主表保持下架状态（`disabledFlag=true`）
- 提示："草稿已删除"

**选项 2：删除草稿并恢复上线**
- 删除草稿记录
- 主表恢复上线（`disabledFlag=false`）
- 清理缓存
- 提示："草稿已删除，内容已恢复上线"

---

## 四、UI/UX 设计细节

### 4.1 提示条（Alert）设计

**已发布详情页（只读模式）：**
```jsx
<Alert
  type="info"
  message="此内容已发布。若需修改，请点击'编辑'按钮。注意：修改后需要重新提交审核才能发布。"
  showIcon
  closable={false}
/>
```

**编辑模式（草稿状态）：**
```jsx
<Alert
  type="warning"
  message="当前为草稿状态，需要提交审核后才能发布。"
  showIcon
  closable={false}
/>
```

### 4.2 确认弹窗设计

**编辑确认弹窗：**
```jsx
modal.confirm({
  title: t('editor.confirmEditTitle'),
  content: t('editor.confirmEditContent'),
  okText: t('editor.confirmEdit'),
  cancelText: t('common.cancel'),
  onOk: async () => {
    // 调用后端接口
    const { draftId } = await apiEditPublished(id)
    // 切换到编辑模式
    setMode('draft')
    setDraftId(draftId)
    // 重新加载草稿数据
    await loadDraft(draftId)
  }
})
```

**批量下架确认弹窗：**
```jsx
modal.confirm({
  title: t('list.confirmBatchOfflineTitle'),
  content: t('list.confirmBatchOfflineContent', { count: selectedIds.length }),
  okText: t('list.confirmOffline'),
  cancelText: t('common.cancel'),
  onOk: async () => {
    await apiBatchOffline(selectedIds)
    message.success(t('list.batchOfflineSuccess', { count: selectedIds.length }))
    navigate('/products?tab=unpublished')
  }
})
```

**删除草稿确认弹窗：**
```jsx
modal.confirm({
  title: t('draft.confirmDeleteTitle'),
  content: t('draft.confirmDeleteContent'),
  okText: t('draft.deleteAndRestore'),
  cancelText: t('draft.deleteOnly'),
  onOk: async () => {
    // 删除草稿 + 恢复上线
    await apiDeleteDraft(draftId)
    await apiRestoreOnline(productId)
    message.success(t('draft.deleteAndRestoreSuccess'))
    navigate('/products?tab=published')
  },
  onCancel: async () => {
    // 仅删除草稿
    await apiDeleteDraft(draftId)
    message.success(t('draft.deleteSuccess'))
    navigate('/products?tab=unpublished')
  }
})
```

### 4.3 按钮位置

**详情页右上角：**
- 只读模式：`[编辑]` 按钮（替代原来的"保存"按钮位置）
- 编辑模式：底部操作栏 `[保存草稿]` `[提交审核]`

**列表页右上角：**
- 操作下拉菜单：
  - 批量删除
  - 批量下架

---

## 五、技术实现方案

### 5.1 后端接口设计

#### 新增接口：编辑已发布内容

**产品：**
```
POST /product/edit-published/{id}
```

**请求参数：** 无（只需要 productId）

**响应：**
```json
{
  "code": 0,
  "ok": true,
  "data": 123,  // draftId
  "msg": "success"
}
```

**后端逻辑：**
```java
@Transactional(rollbackFor = Exception.class)
public ResponseDTO<Long> editPublished(Long productId) {
    RequestUser requestUser = SmartRequestUtil.getRequestUser();
    
    // 1. 权限校验
    ProductEntity product = productDao.selectById(productId);
    if (!product.getOwnerUserId().equals(requestUser.getUserId())) {
        return ResponseDTO.userErrorParam("无权操作此产品");
    }
    
    // 2. 状态校验
    if (!Integer.valueOf(1).equals(product.getStatus())) {
        return ResponseDTO.userErrorParam("只有已发布的产品才能编辑");
    }
    
    // 3. 防止重复创建草稿
    ProductDraftEntity existing = productDraftDao.getLatestDraftByProductId(productId);
    if (existing != null && !existing.getDraftStatus().equals(DraftStatusEnum.REJECTED.getValue())) {
        return ResponseDTO.userErrorParam("该产品已有草稿，请直接编辑现有草稿");
    }
    
    // 4. 主表下架
    product.setDisabledFlag(true);
    productDao.updateById(product);
    productManager.removeCache(productId);
    productManager.removeDetailCache(productId);
    
    // 5. 创建草稿
    ProductDraftEntity draft = buildDraftFromProduct(product, requestUser);
    draft.setDraftStatus(DraftStatusEnum.DRAFT.getValue());
    draft.setActionType(2);  // 修改
    productDraftDao.insert(draft);
    
    log.info("已发布产品转草稿, productId: {}, draftId: {}, operator: {}",
            productId, draft.getDraftId(), requestUser.getUserName());
    
    return ResponseDTO.ok(draft.getDraftId());
}
```

#### 新增接口：批量下架

**产品：**
```
POST /product/batch-offline
```

**请求参数：**
```json
{
  "productIds": [1, 2, 3]
}
```

**响应：**
```json
{
  "code": 0,
  "ok": true,
  "data": {
    "successCount": 3,
    "failCount": 0
  },
  "msg": "success"
}
```

### 5.2 前端状态管理

**ProductEditor 组件的三种模式：**
```typescript
type EditorMode = 'create' | 'draft' | 'detail'

// create: 新建产品
// draft: 编辑草稿（包括新建的草稿和从已发布转换的草稿）
// detail: 查看已发布产品（只读）
```

**模式切换逻辑：**
```typescript
// 初始加载
useEffect(() => {
  if (mode === 'detail' && paramId) {
    loadPublishedProduct(paramId)
  } else if (mode === 'draft' && draftId) {
    loadDraft(draftId)
  }
}, [mode, paramId, draftId])

// 点击编辑按钮
const handleEdit = async () => {
  const confirmed = await modal.confirm({...})
  if (!confirmed) return
  
  const { draftId } = await apiEditPublished(paramId)
  setMode('draft')
  setDraftId(draftId)
  await loadDraft(draftId)
}
```

### 5.3 四个维度统一实现

**接口命名规范：**
```
POST /product/edit-published/{id}
POST /store/edit-published/{id}
POST /brand/edit-published/{id}
POST /agronomist/edit-published/{id}

POST /product/batch-offline
POST /store/batch-offline
POST /brand/batch-offline
POST /agronomist/batch-offline
```

**Service 层统一逻辑：**
1. 权限校验（归属人校验）
2. 状态校验（只有已发布才能编辑）
3. 防止重复创建草稿
4. 主表下架（`disabledFlag=true`）
5. 创建草稿（从主表复制数据）
6. 清理缓存

**前端组件统一：**
- ProductEditor
- StoreEditor
- BrandEditor
- AgronomistEditor

所有 Editor 组件都遵循相同的模式切换逻辑和 UI 设计。

---

## 六、i18n 文案

### 6.1 中文（zh-CN.json）

```json
{
  "editor": {
    "publishedAlert": "此内容已发布。若需修改，请点击'编辑'按钮。注意：修改后需要重新提交审核才能发布。",
    "draftAlert": "当前为草稿状态，需要提交审核后才能发布。",
    "confirmEditTitle": "确认编辑",
    "confirmEditContent": "编辑已发布内容需要重新提交审核。确认要编辑吗？",
    "confirmEdit": "确认编辑",
    "editButton": "编辑",
    "saveDraft": "保存草稿",
    "submitAudit": "提交审核",
    "draftSaved": "草稿已保存",
    "submitSuccess": "已提交审核"
  },
  "list": {
    "batchOffline": "批量下架",
    "confirmBatchOfflineTitle": "确认批量下架",
    "confirmBatchOfflineContent": "下架后将变为草稿状态，需要重新提交审核才能发布。确认下架 {{count}} 个内容吗？",
    "confirmOffline": "确认下架",
    "batchOfflineSuccess": "{{count}} 个内容已下架并转为草稿"
  },
  "draft": {
    "confirmDeleteTitle": "确认删除草稿",
    "confirmDeleteContent": "删除草稿后，是否恢复内容上线？",
    "deleteAndRestore": "删除草稿并恢复上线",
    "deleteOnly": "仅删除草稿",
    "deleteAndRestoreSuccess": "草稿已删除，内容已恢复上线",
    "deleteSuccess": "草稿已删除"
  }
}
```

### 6.2 英文（en.json）

```json
{
  "editor": {
    "publishedAlert": "This content is published. To edit, click the 'Edit' button. Note: Changes require re-submission for review.",
    "draftAlert": "This is a draft. Submit for review to publish.",
    "confirmEditTitle": "Confirm Edit",
    "confirmEditContent": "Editing published content requires re-submission for review. Confirm?",
    "confirmEdit": "Confirm Edit",
    "editButton": "Edit",
    "saveDraft": "Save Draft",
    "submitAudit": "Submit for Review",
    "draftSaved": "Draft saved",
    "submitSuccess": "Submitted for review"
  },
  "list": {
    "batchOffline": "Batch Offline",
    "confirmBatchOfflineTitle": "Confirm Batch Offline",
    "confirmBatchOfflineContent": "Content will become drafts and require re-submission. Offline {{count}} items?",
    "confirmOffline": "Confirm Offline",
    "batchOfflineSuccess": "{{count}} items offlined and converted to drafts"
  }
}
```

### 6.3 西班牙语（es-ES.json）

```json
{
  "editor": {
    "publishedAlert": "Este contenido está publicado. Para editar, haga clic en 'Editar'. Nota: Los cambios requieren nueva revisión.",
    "draftAlert": "Este es un borrador. Envíe para revisión para publicar.",
    "confirmEditTitle": "Confirmar Edición",
    "confirmEditContent": "Editar contenido publicado requiere nueva revisión. ¿Confirmar?",
    "confirmEdit": "Confirmar Edición",
    "editButton": "Editar",
    "saveDraft": "Guardar Borrador",
    "submitAudit": "Enviar para Revisión",
    "draftSaved": "Borrador guardado",
    "submitSuccess": "Enviado para revisión"
  },
  "list": {
    "batchOffline": "Desconectar en Lote",
    "confirmBatchOfflineTitle": "Confirmar Desconexión en Lote",
    "confirmBatchOfflineContent": "El contenido se convertirá en borradores y requerirá nueva revisión. ¿Desconectar {{count}} elementos?",
    "confirmOffline": "Confirmar Desconexión",
    "batchOfflineSuccess": "{{count}} elementos desconectados y convertidos en borradores"
  }
}
```

---

## 七、测试用例

### 7.1 详情页编辑流程

**TC-1：BC 用户编辑已发布产品**
1. BC 用户登录
2. 打开已发布产品详情页
3. 验证：顶部显示提示条，右上角有"编辑"按钮
4. 点击"编辑"按钮
5. 验证：弹出确认弹窗
6. 点击"确认编辑"
7. 验证：页面进入编辑模式，顶部提示条变为警告类型
8. 修改产品信息
9. 点击"保存草稿"
10. 验证：草稿已保存，停留在当前页面
11. 点击"提交审核"
12. 验证：跳转到"草稿"列表，提示"已提交审核"

**TC-2：ADMIN 用户无法使用编辑功能**
1. ADMIN 用户登录
2. 打开已发布产品详情页
3. 验证：不显示"编辑"按钮（ADMIN 直接编辑主表，不需要这个功能）

**TC-3：防止重复创建草稿**
1. BC 用户编辑已发布产品，创建草稿
2. 返回产品详情页
3. 再次点击"编辑"按钮
4. 验证：提示"该产品已有草稿，请直接编辑现有草稿"

### 7.2 批量下架流程

**TC-4：批量下架产品**
1. BC 用户登录
2. 在"已发布"列表中选中 3 个产品
3. 点击右上角操作下拉 → 批量下架
4. 验证：弹出确认弹窗，显示"确认下架 3 个内容吗？"
5. 点击"确认下架"
6. 验证：跳转到"草稿"列表，提示"3 个内容已下架并转为草稿"
7. 验证：草稿列表中出现 3 个新草稿，`actionType=2`（修改）

### 7.3 四个维度统一测试

**TC-5：店铺编辑流程**
- 重复 TC-1，但使用店铺维度
- 验证：流程和产品完全一致

**TC-6：品牌编辑流程（免审核）**
- COMPANY 用户，品牌已认证
- 编辑已发布品牌，提交审核
- 验证：自动审核通过并发布（免审核直发）

**TC-7：农艺师编辑流程（免审核）**
- COMPANY 用户，品牌已认证
- 编辑已发布农艺师，提交审核
- 验证：自动审核通过并发布（免审核直发）

### 7.4 边界情况测试

**TC-8：编辑草稿状态的内容**
1. 打开草稿详情页
2. 验证：不显示"编辑"按钮（已经是草稿，直接编辑即可）

**TC-9：编辑待审核状态的内容**
1. 打开待审核内容详情页
2. 验证：不显示"编辑"按钮，所有字段只读

**TC-10：编辑已拒绝状态的内容**
1. 打开已拒绝内容详情页
2. 验证：不显示"编辑"按钮（已经是草稿，直接编辑即可）

---

## 八、实施计划

### 8.1 开发任务分解

**后端任务（backend-dev）：**
1. 实现 `editPublished()` 方法（四个维度）
2. 实现 `batchOffline()` 方法（四个维度）
3. 修改现有的 `offline()` 方法，增加创建草稿逻辑
4. 添加详细日志

**前端任务（frontend-dev）：**
1. 修改 Editor 组件，增加"编辑"按钮和提示条
2. 实现模式切换逻辑（detail → draft）
3. 修改列表页，增加"批量下架"操作
4. 添加确认弹窗和提示信息
5. i18n 三语言同步

**测试任务（e2e-tester）：**
1. 编写 E2E 测试用例（TC-1 ~ TC-10）
2. 四个维度全覆盖测试
3. 边界情况测试

**审查任务（reviewer）：**
1. 代码审查（安全性、规范性）
2. 业务逻辑审查（是否符合需求）
3. UI/UX 审查（是否符合设计）

### 8.2 实施顺序

**Phase 1：产品维度实现（验证方案）**
1. 后端实现产品维度的接口
2. 前端实现产品维度的 UI
3. E2E 测试验证
4. 用户验收

**Phase 2：扩展到其他维度**
1. 复制产品维度的实现到店铺/品牌/农艺师
2. 调整免审核逻辑
3. E2E 全覆盖测试

**Phase 3：清理旧代码**
1. 删除 v2.4 的"保存到草稿"/"提交审核"按钮
2. 统一所有 Editor 组件的交互逻辑
3. 更新文档

### 8.3 风险评估

**风险 1：用户不理解需要重新审核**
- 缓解措施：多层提示（提示条、确认弹窗、帮助文档）
- 监控指标：用户反馈、客服工单

**风险 2：批量下架误操作**
- 缓解措施：确认弹窗明确说明后果
- 监控指标：批量下架操作的撤销率

**风险 3：性能问题（批量下架大量内容）**
- 缓解措施：限制批量操作的数量（如最多 50 个）
- 监控指标：接口响应时间

---

## 九、后续优化方向

### 9.1 短期优化（v1.1）

1. **撤销下架功能**：允许用户在一定时间内撤销下架操作
2. **批量操作进度条**：批量下架时显示进度
3. **草稿自动保存**：编辑时自动保存草稿，避免数据丢失

### 9.2 长期优化（v2.0）

1. **版本对比**：显示草稿和已发布版本的差异
2. **协作编辑**：多人同时编辑同一内容的冲突处理
3. **审核流程优化**：支持多级审核、审核意见反馈

---

## 十、附录

### 10.1 相关文档

- `docs/business/workflow-draft.md` - 草稿状态机规范
- `docs/business/online-offline.md` - 上下架语义规范
- `docs/frontend/editor-architecture.md` - Editor 架构规范
- `EDITOR_REFACTOR_PLAN.md` - Editor 规范统一计划

### 10.2 数据库表结构

**主表字段：**
- `disabled_flag` - 上下架标志（true=下架，false=在线）
- `status` - 发布状态（0=未发布，1=已发布，2=已拒绝）

**草稿表字段：**
- `draft_status` - 草稿状态（0=草稿，1=待审核，2=已拒绝）
- `action_type` - 操作类型（1=新增，2=修改）
- `product_id` - 关联的主表 ID

### 10.3 免审核逻辑

**品牌/店铺/农艺师：**
- 条件：COMPANY 用户 + 品牌已认证
- 行为：提交审核时自动审核通过并发布

**产品：**
- 保留人工审核流程
- 不支持免审核直发

---

**文档结束**
