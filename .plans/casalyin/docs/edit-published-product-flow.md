# 编辑已发布产品功能流程总结

> 创建时间：2026-04-25
> 适用维度：产品、品牌、店铺、农艺师

---

## 功能概述

用户可以编辑已发布的内容，修改后通过草稿机制提交审核（产品）或直接发布（其他维度）。主表数据在审核通过前保持稳定。

---

## 完整流程

### 1. 进入编辑模式

```
用户点击已发布内容 → 进入详情页（mode=detail，可编辑）
  ↓
前端检查是否已有草稿
  ├─ 有草稿：优先加载草稿数据（draftStatus=0/2）
  └─ 无草稿：加载主表数据
```

**关键接口：**
- `GET /{dimension}/detail/{id}` — 获取主表数据
- `GET /{dimension}/draft/my` — 检查是否有草稿（按 entityId 过滤）

### 2. 保存草稿（不提交审核）

```
用户修改后点击"保存草稿"
  ↓
检查是否已有草稿
  ├─ 没有草稿：POST /{dimension}/draft/create（actionType=2，entityId=主表ID）
  └─ 有草稿：POST /{dimension}/draft/update（更新草稿）
  ↓
draftStatus = 0（草稿状态）
主表不变
```

**关键点：**
- `actionType=2` 表示"编辑已发布内容"
- `entityId` 指向主表记录的 ID
- 草稿可以多次保存，不需要立即提交

### 3. 提交审核/发布

```
用户修改后点击"提交审核"（产品）或"发布"（其他维度）
  ↓
检查是否已有草稿
  ├─ 没有草稿：创建草稿（复制主表数据 + 用户修改）
  └─ 有草稿：更新草稿
  ↓
POST /{dimension}/draft/submit
  ↓
  ├─ 产品：draftStatus=1（待审核），需要 Admin 审核
  └─ 其他维度：违规词检测
      ├─ 命中 HARD 词 → 返回错误，草稿保留
      ├─ 命中 SOFT 词 → COMPANY 例外直接发布，其他角色转人工审核
      └─ 无命中 → 免审直发（更新主表 + 记录变更日志 + 删除草稿）
```

**关键接口：**
- `POST /{dimension}/draft/submit` — 提交审核/发布

### 4. 审核通过（仅产品）

```
Admin 审核通过
  ↓
用草稿数据更新主表
  ↓
删除草稿
  ↓
通知用户（DRAFT_APPROVED）
```

**关键接口：**
- `POST /{dimension}/draft/approve/{draftId}` — Admin 审核通过

---

## 关键数据结构

### 草稿表字段

| 字段 | 说明 |
|------|------|
| id | 草稿 ID |
| entityId | 主表记录 ID（编辑已发布时必填） |
| actionType | 1=新建，2=编辑已发布 |
| draftStatus | 0=草稿，1=待审核，2=已拒绝 |
| userId | 创建者 ID |
| ownerUserId | 所有者 ID（品牌申请通过后填充） |

### 内容变更日志（免审直发时记录）

| 字段 | 说明 |
|------|------|
| entityType | 1=产品，2=品牌，3=店铺，4=农艺师 |
| entityId | 主表记录 ID |
| actionType | 1=新建，2=更新 |
| diffJson | 变更字段 JSON：`[{field, label, before, after}]` |
| operatorId | 操作者 ID |

---

## 前端关键组件

### ProductEditor（以产品为例）

```tsx
mode: 'create' | 'draft' | 'detail'

// detail 模式下的逻辑
useEffect(() => {
  if (mode === 'detail' && id) {
    // 1. 先检查是否有草稿
    checkDraft(id).then(draft => {
      if (draft && [0, 2].includes(draft.draftStatus)) {
        // 有草稿：加载草稿数据
        loadDraft(draft.id)
      } else {
        // 无草稿：加载主表数据
        loadPublished(id)
      }
    })
  }
}, [mode, id])

// 保存草稿
const handleSaveDraft = () => {
  if (draftId) {
    // 更新草稿
    updateDraft(draftId, formData)
  } else {
    // 创建草稿（actionType=2, entityId=主表ID）
    createDraft({ ...formData, actionType: 2, entityId: id })
  }
}

// 提交审核
const handleSubmit = () => {
  if (draftId) {
    // 提交已有草稿
    submitDraft(draftId)
  } else {
    // 创建草稿并提交
    createDraft({ ...formData, actionType: 2, entityId: id })
      .then(draft => submitDraft(draft.id))
  }
}
```

---

## 测试场景

### 场景 1：编辑已发布产品（无草稿）

1. BASE 用户创建产品并发布（审核通过）
2. 用户点击已发布产品进入详情页
3. 修改产品名称
4. 点击"保存草稿" → 创建草稿（actionType=2）
5. 验证：草稿表有记录，draftStatus=0，entityId=主表ID
6. 验证：主表数据未变

### 场景 2：编辑已发布产品（有草稿）

1. 用户已有草稿（draftStatus=0）
2. 用户再次进入详情页
3. 验证：加载的是草稿数据，不是主表数据
4. 修改产品描述
5. 点击"保存草稿" → 更新草稿
6. 验证：草稿更新成功，主表数据未变

### 场景 3：提交审核并通过

1. 用户编辑已发布产品并保存草稿
2. 点击"提交审核" → draftStatus=1
3. Admin 审核通过
4. 验证：主表数据已更新为草稿数据
5. 验证：草稿已删除
6. 验证：用户收到 DRAFT_APPROVED 通知

### 场景 4：品牌免审直发

1. COMPANY 用户编辑已发布品牌
2. 修改品牌描述（无违规词）
3. 点击"发布"
4. 验证：主表立即更新
5. 验证：草稿已删除
6. 验证：t_content_change_log 有记录（actionType=2，diffJson 包含变更字段）

### 场景 5：违规词拦截

1. COMPANY 用户编辑已发布店铺
2. 修改店铺名称，包含 HARD 违规词
3. 点击"发布"
4. 验证：返回错误提示
5. 验证：草稿保留（draftStatus=0）
6. 验证：主表数据未变

---

## 已知问题与修复

### Bug 1：草稿审核通过后 ownerUserId 为 null

**问题：** 品牌申请通过后，产品的 ownerUserId 未填充，导致 COMPANY 用户无法编辑产品。

**修复：** `BrandDraftServiceImpl.approveDraft()` 中，审核通过后调用 `ProductService.syncCreatorToBrandOwner()` 同步产品所有者。

**测试：** `casalyin-server/e2e/v2.5/edit-published-product.spec.ts`

---

## 相关文档

- `docs/business/workflow-draft.md` — 草稿审核流程与状态规则
- `docs/backend/api-conventions.md` — 接口规范
- `docs/frontend/editor-architecture.md` — XxxEditor 三合一架构
