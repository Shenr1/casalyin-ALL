# 二次审查报告 — Task #4 ContentDetailLayout 底部操作栏统一改造

**日期**：2026-04-14  
**审查人**：reviewer  
**请求方**：frontend-dev  
**基准报告**：report.md（[BLOCK] 2×HIGH + 2×MEDIUM + 1×LOW）

---

## 最终结论

✅ **[OK]** — 全部 HIGH 问题已修复，MEDIUM/LOW 问题亦已修复或妥善处理，审查通过。

---

## 逐项修复验证

### HIGH-1：ContentDetailLayout 10+ 处硬编码中文 ✅ 已修复

- `useTranslation` 已引入（L17-18），`const { t } = useTranslation()`（L88）
- 9 处文案全部替换为 `t('contentDetailLayout.*')`：
  - L113 noPreview ✅ / L145 metaTitle ✅ / L165 opsTitle ✅
  - L171 certify/uncertify ✅ / L178 online ✅ / L183 offline ✅
  - L190 sendNotice ✅ / L199 delete ✅
- 三语言 key 核查：
  - zh-CN L461-470：9 key 全部存在 ✅
  - en L1106-1115：9 key 全部存在 ✅
  - es-ES L780-789：9 key 全部存在 ✅
- Header 中原「保存」「保存并发布」按钮已从渲染层移除（函数体不再解构 onSave/onSaveAndPublish）✅
- **结论：通过**

---

### HIGH-2：AgronomistEditor detail 模式 BC 用户 StickyFooterBar 无操作按钮 ✅ 已修复

- 新增 `handleDetailSaveDraft`（L457-476）：调用 `apiAgronomistDraftAdd(buildPayload(values))`，成功后跳转 `/agronomists/edit-draft/${draftId}`
- 后端 `addDraft` Service 逻辑验证：根据 `agronomistDao.getByUserId(userId)` 自动判断 actionType（1=新建/2=修改已有农艺师），自动关联 `agronomistId`，前端无需传 agronomistId，方案正确 ✅
- `onSave` 赋值逻辑（L629-632）：`detail && !isAdmin` 分支现在赋值 `handleDetailSaveDraft`，BC 用户 FooterBar 右侧有「保存」按钮 ✅
- **结论：通过**

---

### MEDIUM-2：Header「保存并发布」无效按钮 ✅ 已修复

- `onSaveAndPublish` / `onSave` / `publishLabel` / `saving` / `publishing` 标注 `@deprecated`，仅保留在 interface 类型声明中（L57-67）
- 函数体（L70-87）已不解构这些字段，渲染层完全不渲染旧 Header 按钮 ✅
- **结论：通过**

---

### MEDIUM-1：BrandEditor.handleDetailSaveDraft 缺 setLastSavedAt

成功后立即 navigate 跳走，体验无影响。上次标注为一致性建议，本次未修复，可后续统一清理，不阻断。

### LOW-1：StickyFooterBar 重复 import dayjs locale

无害，未修复，可后续整理，不阻断。

---

## 新发现问题

无。

---

## 通过项汇总

| 检查项 | 结果 |
|--------|------|
| ContentDetailLayout 全部文案走 t() | ✅ |
| contentDetailLayout.* 9 key 三语言同步 | ✅ |
| Header 旧按钮已从渲染层移除 | ✅ |
| AgronomistEditor handleDetailSaveDraft 实现正确 | ✅ |
| 后端 addDraft 自动关联 agronomistId（无需前端传参） | ✅ |
| BC 用户 detail 模式 FooterBar 有操作按钮 | ✅ |
| @deprecated 字段仅保留类型声明，不参与渲染 | ✅ |
| 无 Modal.confirm / message.success 静态方法 | ✅ |
| TypeScript 编译 0 错误 | ✅（frontend-dev 自验） |
