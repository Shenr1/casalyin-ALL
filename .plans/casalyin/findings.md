# casalyin - 团队发现记录

> 纯索引——每个条目简短（Status + 来源 + 关键结论）。详细内容见各智能体子目录。

---

## E2E 测试关键发现

### F1 — 列表页宽度（已决策）
**来源：** 第六期 UI 规范回归（TC-UI-001）
**发现：** 列表页实际 maxWidth=1440px（ProLayout 默认），与旧规范 1000px 不一致。
**决策：** 用户选择 option 2 — 改规范，不改代码。`docs/frontend/ui-components.md` 已更新（2026-04-12）。
**Status: resolved**

### F2 — BASE 无法访问 /brands/list（预期行为）
**来源：** 第六期 UI 规范回归
**发现：** BASE 角色访问 `/brands/list` 被拦截，因为该路由需要 `brand:update/add/delete` 权限。
**结论：** 预期行为，不是 Bug。BASE 只能通过产品/店铺/农艺师页面间接关联品牌。
**Status: confirmed, no action**

### F3 — Ant Design `modal.confirm()` 不生成 role="dialog"
**来源：** 第七期端到端测试（TC-E2E-001）
**发现：** `AppProvider` 导出的 `modal.confirm()` 生成的是 `.ant-modal-confirm` 类元素，而不是标准 `role="dialog"` 元素，导致 `getByRole('dialog')` 无法匹配。
**正确选择器：** `.ant-modal-confirm:visible` / `.ant-modal-confirm-btns .ant-btn-primary`
**Status: documented（已写入 e2e-tester/progress.md）**

### F4 — 品牌维度无上下架功能
**来源：** 第五期上下架专项
**发现：** 品牌维度没有 onOnline/onOffline 处理器，后端也无对应端点。
**结论：** 设计如此，品牌无上下架概念。三个维度（产品/店铺/农艺师）均有。
**Status: confirmed, no action**

---

## 品牌认证体系

### 第一期（后端）✅
- Flyway V4 迁移：`t_store.brand_id`、`t_agronomist.brand_id`、`t_product.brand_owner_id`
- COMPANY 创建内容时自动写入 brand_id
- COMPANY 提交草稿时跳过审核直接发布（D10：SOFT 违规词也跳过）

### 第二期（前端）✅
- StoreEditor / ProductEditor / AgronomistEditor：COMPANY 品牌字段预填只读
- Bug 11 Fix：`Form.Item` 去掉 `name="brandId"` prop，避免表单值覆盖 `value` prop

### 第三期（品牌认领批量回写）⏳ pending
- 品牌认领时批量写入现有产品的 `brandOwnerId`（B-品牌3）

---

## 待开发 Backlog（优先级排序）

| # | 内容 | 优先级 |
|---|------|--------|
| B1 | 产品 `submitDraft` 缺敏感词检测 | 中 |
| B3 | D9 平台认证申请体系 + ContentStatusCard | 中 |
| B-品牌3 | 品牌认领 → 批量写入 brandOwnerId | 中 |
| B2 | `isCompanyUser()` 抽离为 RoleUtils | 低 |
| B4 | i18n 补全 en/es-ES | 低 |

详见 `.plans/casalyin/decisions.md` 中的 Backlog 表格。
