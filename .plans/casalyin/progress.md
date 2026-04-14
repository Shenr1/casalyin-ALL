# casalyin - 工作日志

> 按时间线记录。

---

## 2026-04-11 Session 1 — 团队搭建

### 已完成
- [x] 将所有规范从 Claude 全局 memory 迁移到 docs/ 目录（12 个文件）
- [x] 创建项目 CLAUDE.md（索引 + 核心约定）
- [x] 创建 .plans/casalyin/ 目录结构
- [x] 搭建团队（frontend-dev、backend-dev、e2e-tester、reviewer）

---

## 2026-04-11 Session 2 — 第一期 E2E + Bug 修复

### 已完成
- [x] 第一期 E2E：认证与会话管理，16/16 PASS（auth/login.spec.ts）
- [x] 第二期 E2E：草稿状态机，10/10 PASS（draft/product-draft.spec.ts）
  - Bug 1-7 全部修复（权限码、静态 message、setCreatorType、rejectReason、权限校验、PENDING disabled）
- [x] reviewer 审查通过：PermissionService 权限码变更 [OK]

---

## 2026-04-12 Session 3 — 第三期 + 品牌认证 + 上下架

### 已完成
- [x] 第三期 E2E：已发布内容状态管理，4/4 PASS（publish/product-publish.spec.ts）
  - Bug 8 修复：apiCertifyProduct 参数嵌套错误
  - Bug 9 修复：ContentDetailLayout 上架按钮显示条件（四维度共享，一次修复）
- [x] 品牌认证体系后端（第一期）：
  - Flyway V4 迁移（brand_id 字段）
  - COMPANY 免审直发（含 D10：SOFT 违规词也跳过）
  - 自动写入 brand_id
- [x] 品牌认证体系前端（第二期）：
  - COMPANY 品牌字段预填只读
  - Bug 11 修复：Form.Item 去掉 name prop
  - i18n 三语言补全
- [x] 第 3.5 期 E2E：品牌认证回归，7/7 PASS（company/brand-certification.spec.ts）
  - Bug 10 修复：农艺师草稿创建失败
- [x] 第五期 E2E：上下架专项，4/4 PASS（online/online-offline.spec.ts）
  - 确认品牌无上下架功能（设计如此）
- [x] 第六期 E2E：UI 规范全站回归，13/14 PASS（ui/ui-standards.spec.ts）
  - F1 页面宽度：用户决策 option 2，规范从 1000px 改为 ProLayout 默认 1440px
  - docs/frontend/ui-components.md 已更新
- [x] 第七期 E2E：端到端完整业务场景，1/1 PASS（e2e/full-lifecycle.spec.ts）
  - 全链路验证：BASE 创建→提交→ADMIN 审核→发布→下架→上架，39.2s 完成
  - 关键经验：modal.confirm() 使用 .ant-modal-confirm:visible 选择器

### 当前状态
七期测试全部完成，11 个 Bug 已修复，等待用户决定 Backlog 方向。
