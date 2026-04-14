# 代码审查：B3-3 — 用户品牌认证申请入口页面

> 审查人: reviewer
> 日期: 2026-04-12
> 目标: BrandApplyMyPage 新建 + 路由注册 + i18n 三语言 + routing.md 更新

## 变更清单

| # | 文件 | 变更类型 |
|---|------|----------|
| 1 | `casalyin-server/src/pages/BrandApplyMyPage.tsx` | 新建（~610 行） |
| 2 | `casalyin-server/src/routes/index.tsx` | 新增路由 `/brands/apply-my` |
| 3 | `casalyin-server/src/locales/zh-CN.json` | 新增 `brandApplyMy` 60 个 key |
| 4 | `casalyin-server/src/locales/en.json` | 新增 `brandApplyMy` 60 个 key |
| 5 | `casalyin-server/src/locales/es-ES.json` | 新增 `brandApplyMy` 60 个 key |
| 6 | `docs/frontend/routing.md` | 新增路由条目 |

---

## 审查结论：[OK] 通过

无 CRITICAL 或 HIGH 问题。1 个 LOW 级别建议。

---

## 项目规范检查

### 前端核心禁忌

| 检查项 | 结果 | 备注 |
|--------|------|------|
| 禁止 `Modal.confirm` 静态方法 | ✅ 合规 | 使用 AppProvider 导出的 `modal.confirm`（第 258 行） |
| 禁止 `message.success` 静态方法 | ✅ 合规 | 使用 AppProvider 导出的 `message.success`（第 73/270 行） |
| `<Modal>` 声明式 JSX 弹窗 | ✅ 合规 | `AntModal` 是 `antd` 的 `Modal` 组件别名，用于表单弹窗，声明式用法合规 |
| 禁止硬编码文案 | ✅ 合规 | 所有中文仅出现在注释中，用户可见文案全部通过 `t()` |
| i18n 三语言同步 | ✅ 合规 | zh-CN / en / es-ES 均为 60 个 key，完全对齐 |
| 禁止独立 XxxCreate/XxxDetail | ✅ N/A | 非编辑器页面，是状态展示页 |
| ContentDetailLayout | ✅ N/A | 非编辑器/详情页，状态展示页不适用 |
| 返回按钮规范（仅箭头图标） | ✅ 合规 | 第 592 行 `<Button icon={<ArrowLeftOutlined />}>` 无文字 |
| 页面宽度 | ✅ 合规 | `maxWidth: 900`（第 588 行），符合规范 |

### 路由

| 检查项 | 结果 |
|--------|------|
| 路由已注册 | ✅ `/brands/apply-my`（index.tsx 第 489-494 行） |
| PrivateRoute 包裹 | ✅ |
| routing.md 已更新 | ✅ 第 91 行 |
| 路由路径 kebab-case | ✅ `/brands/apply-my` |

---

## 详细审查

### 1. BrandApplyMyPage.tsx

**架构**：组件拆分清晰，采用 4 个子组件（NoApplyView / PendingView / ApprovedView / RejectedView）+ 1 个 ApplyModal + 1 个主入口组件。职责分明。

**Modal 使用方式**：
- 表单弹窗使用 `<AntModal>`（JSX 声明式）— 正确做法
- 确认弹窗使用 `modal.confirm`（AppProvider 导入的实例）— 正确做法
- 两种用法均合规

**防重复提交**：`submittingRef.current` 锁 + `setLoading(true)` 双保险（第 55-83 行），可靠。

**状态流转**：
- 无申请/已撤销(3) → NoApplyView → 申请弹窗
- PENDING(0) → PendingView → 撤回
- APPROVED(1) → ApprovedView（自动刷新用户角色）
- REJECTED(2) → RejectedView → 重新申请弹窗

覆盖完整，与后端 status 定义一致。

**角色刷新**：ApprovedView 触发 `refreshUser()`（第 557-560 行），确保前端角色信息及时同步。

### 2. routes/index.tsx

- `/brands/apply-my` 放在品牌模块路由区域内（第 489-494 行），位置合理
- 无需 PermissionRoute（BASE/COMPANY 用户都可访问），正确

### 3. i18n 文件

- 命名空间 `brandApplyMy` 清晰，不与已有 `brandApplyAdmin` 冲突
- 60 个 key 全覆盖组件中所有 `t()` 调用
- 三语言翻译质量看起来合理（en 自然通顺，es-ES 语法正确）

---

## 发现问题

### LOW-1：`useEffect` 依赖项可补全

**位置**：BrandApplyMyPage.tsx 第 556-560 行
```tsx
useEffect(() => {
  if (applyRecord?.status === 1) {
    refreshUser()
  }
}, [applyRecord?.status])
```
**描述**：ESLint exhaustive-deps 规则可能会提示缺少 `refreshUser` 依赖。如果 `refreshUser` 是稳定引用（useCallback/ref），则无影响。
**级别**：LOW — 不影响功能，仅代码质量建议

---

## 总结

新建的 BrandApplyMyPage 质量高，架构清晰，完全遵循项目规范。Modal/message 使用方式正确（声明式 JSX + AppProvider 实例），所有文案走 i18n，三语言同步完整。路由注册和文档更新到位。

**审查结果：[OK] 通过** — 可合并。
