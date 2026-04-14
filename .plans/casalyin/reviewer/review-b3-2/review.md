# 代码审查：B3-2 — 品牌申请管理页面规范修复

> 审查人: reviewer
> 日期: 2026-04-12
> 目标: BrandApplyManagement + BrandApplyDetail 规范违规修复 + i18n 三语言

## 变更清单

| # | 文件 | 变更类型 |
|---|------|----------|
| 1 | `casalyin-server/src/pages/BrandApplyManagement.tsx` | 重写（message 导入、isAdmin 判断、硬编码→t()、移除 maxWidth） |
| 2 | `casalyin-server/src/pages/BrandApplyDetail.tsx` | 重写（message/modal 导入、isAdmin 判断、40+硬编码→t()、fmt 参数名、移除 maxWidth） |
| 3 | `casalyin-server/src/locales/zh-CN.json` | 新增 `brandApplyAdmin` 44 个 key |
| 4 | `casalyin-server/src/locales/en.json` | 新增 `brandApplyAdmin` 44 个 key |
| 5 | `casalyin-server/src/locales/es-ES.json` | 新增 `brandApplyAdmin` 44 个 key |

---

## 审查结论：[OK] 通过

无 CRITICAL 或 HIGH 问题。无 MEDIUM 问题。

---

## 项目规范检查

### 前端核心禁忌

| 检查项 | 结果 | 备注 |
|--------|------|------|
| 禁止 `message` 从 antd 直接导入 | ✅ 已修复 | Management: `import { message } from '../components/AppProvider'` (L7) |
| | | Detail: `import { modal, message } from '../components/AppProvider'` (L15) |
| 禁止 `Modal.confirm` 静态方法 | ✅ 合规 | Detail 使用 `modal.confirm` (AppProvider 实例，L91/L149) |
| `<Modal>` 声明式 JSX | ✅ 合规 | Detail 拒绝弹窗用 `<Modal>` JSX (L340)，从 antd 导入声明式组件合规 |
| 禁止硬编码文案 | ✅ 已修复 | 所有中文仅出现在注释中 |
| i18n 三语言同步 | ✅ 合规 | zh-CN / en / es-ES 均为 44 个 key，完全对齐 |
| i18n 插值 `{{name}}` | ✅ 合规 | 3 处插值（approveConfirmContent/rejectModalTitle/revokeConfirmContent）三语言一致 |
| `maxWidth: '1000px'` 移除 | ✅ 已移除 | 两个文件均不再包含 maxWidth |

### isAdmin 判断方式

| 检查项 | 结果 | 备注 |
|--------|------|------|
| 旧：`roleTypes.some(...)` | ❌ 已移除 | |
| 新：`permissionList.includes('brand:audit')` | ✅ 正确 | 与后端 PermissionRoute 权限控制一致 |

### fmt 函数参数名

| 检查项 | 结果 |
|--------|------|
| 旧：`const fmt = (t?: string) =>` 遮蔽 useTranslation 的 t | ❌ 已移除 |
| 新：`const fmt = (v?: string) =>` | ✅ 无遮蔽 |

---

## 详细审查

### 1. BrandApplyManagement.tsx

代码干净简洁（122 行），6 个 Table 列标题全部从 `t('brandApplyAdmin.*')` 获取。Card title 和刷新按钮同样走 `t()`。`message.error` 在加载失败时带 fallback `t('common.loadFailed')`，处理合理。

### 2. BrandApplyDetail.tsx

较大页面（366 行），包含：
- **审核操作**：approve/reject，用 `modal.confirm`（AppProvider）+ `<Modal>` 声明式拒绝弹窗
- **订阅管理**：DatePicker + Switch + 保存按钮
- **撤销认证**：`modal.confirm`（AppProvider）+ danger 按钮

40+ 处硬编码全部成功替换为 `t()` 调用。所有错误消息均带 `data?.msg || t('brandApplyAdmin.operationFailed')` fallback，后端错误信息优先展示。

### 3. i18n 文件

44 个 key 完全覆盖两个文件中所有 `t('brandApplyAdmin.*')` 调用。en/es-ES 翻译质量合理，插值参数 `{{name}}` 三语言一致。

---

## 发现问题

无。所有修复项均正确实施。

---

## 总结

本次变更是纯规范修复，将两个历史页面从「antd 静态方法 + 硬编码中文」迁移到「AppProvider 实例 + i18n t()」。所有修复正确到位，无遗漏。`isAdmin` 判断从角色类型检测改为权限码检测，与路由层 PermissionRoute 保持一致性。

**审查结果：[OK] 通过** — 可合并。
