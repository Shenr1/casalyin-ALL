# 代码审查报告 — Task #4 ContentDetailLayout 底部操作栏统一改造

**日期**：2026-04-14  
**审查人**：reviewer  
**请求方**：frontend-dev  
**涉及文件**：
- src/components/StickyFooterBar.tsx（新建）
- src/components/ContentDetailLayout.tsx（新增 footerBar prop）
- src/pages/AgronomistEditor.tsx
- src/pages/StoreEditor.tsx
- src/pages/BrandEditor.tsx
- src/pages/ProductEditor.tsx
- src/locales/zh-CN.json / en.json / es-ES.json

---

## 最终结论

[BLOCK] — 存在 2 处 HIGH 问题，需修复后重新审查。

---

## 问题清单

### HIGH-1：ContentDetailLayout.tsx 内有大量硬编码中文文案（违反 i18n 规范）

**文件**：`src/components/ContentDetailLayout.tsx`

下列文案未走 `t()`：

| 行号 | 硬编码文案 |
|------|-----------|
| L72 | `publishLabel = '保存并发布'`（默认值） |
| L111 | `'暂无预览链接'` |
| L122 | `保存`（Header 保存按钮） |
| L160 | `内容信息`（Sidebar 分区标题） |
| L180 | `操作`（Sidebar 分区标题） |
| L186 | `'取消认证'` / `'认证'` |
| L193 | `上架` |
| L198 | `下架` |
| L205 | `发整改通知` |
| L214 | `删除` |

此组件作为全站四维度共用的骨架组件，硬编码中文会导致多语言场景（en/es-ES）全部显示中文。违反项目规范「禁止硬编码任何用户可见文案」，列为 **HIGH**。

**修复方向**：在 ContentDetailLayout 内部引入 `useTranslation`，所有文案走 `t()`，并在三语言文件中同步添加对应 key。

---

### HIGH-2：AgronomistEditor detail 模式 BC 用户 StickyFooterBar 无任何操作按钮

**文件**：`src/pages/AgronomistEditor.tsx` L594-624

分析：
- `publishAction`（L594-597）：detail + BC 用户 → `undefined`
- `onSave`（L611）：仅 `draft && canEdit` → detail 模式为 `undefined`

结果：BC 用户进入 `/agronomists/:id` 时，StickyFooterBar 右侧完全没有按钮，用户无法保存任何修改（即使表单可编辑）。

农艺师已发布的内容，BC 用户通常需要编辑后再走草稿提交流程。当前 detail 模式 BC 用户缺少「保存为草稿」或「提交修改」的入口，等同于 **功能静默失效**，列为 **HIGH**。

**修复方向**：参照 BrandEditor 的 `handleDetailSaveDraft`，在 AgronomistEditor detail 模式为 BC 用户提供「保存修改（创建草稿）」按钮，并将其接入 StickyFooterBar 的 `onSave` 或 `extraActions`。

---

## MEDIUM 问题

### MEDIUM-1：BrandEditor.handleDetailSaveDraft 缺少 setLastSavedAt

**文件**：`src/pages/BrandEditor.tsx` L372-391

`handleDetailSaveDraft` 保存成功后立即 `navigate` 跳走（L381），不会停留在当前页面，故 `setLastSavedAt` 不影响用户体验。但从代码一致性角度，所有「保存成功」分支都应调用（其他三个 Editor 的同类函数均有调用）。建议添加，避免未来改动跳转逻辑时遗漏。

---

### MEDIUM-2：ContentDetailLayout Header 区域「保存并发布」按钮与 footerBar 功能重叠

**文件**：`src/components/ContentDetailLayout.tsx` L127-134

当前所有 Editor 均传 `onSaveAndPublish={() => {}}`，Header 的「保存并发布」按钮会渲染但点击无效（空函数）。虽然 TS 类型要求 `onSaveAndPublish` 必填，但渲染出一个无效按钮会造成用户困惑。

**修复方向**（二选一）：
- 方案 A：将 `onSaveAndPublish` 改为可选（`onSaveAndPublish?: () => void`），不传时不渲染该按钮
- 方案 B：将其从 Header 移除，统一由 `footerBar` 承载（推荐，与改造目标一致）

---

## LOW 问题

### LOW-1：StickyFooterBar 中 dayjs locale import 与 App.tsx 全局设置存在潜在冲突

**文件**：`src/components/StickyFooterBar.tsx` L15-17 vs `src/App.tsx` L17

`App.tsx` 调用了 `dayjs.locale('zh-cn')` 设置全局默认 locale；`StickyFooterBar` 在实例上调用 `.locale(locale)` 是正确的链式用法，不会污染全局，行为是安全的。

但 `StickyFooterBar` 额外 import 了 `'dayjs/locale/zh-cn'`，而 `App.tsx` 和 `main.tsx` 中已经 import 过。这是无害的重复 import（打包时会去重），但建议整理，只在入口处统一 import。

---

## 通过项

- `StickyFooterBar` 无硬编码文案，全部走 `t()` ✅
- `common.neverSaved / lastSavedAt / justNow` 三语言同步（zh-CN L457-459 / en L1102-1104 / es-ES L776-778）✅
- 无 `Modal.confirm` / `message.success` 静态方法（App.useApp() 解构）✅
- 无独立 XxxCreate/XxxDetail 页面 ✅
- 无 `@SaIgnore` / `@NoNeedLogin` 滥用（前端审查范围内）✅
- `ContentDetailLayout` 新增 `footerBar?: ReactNode`，向后兼容（不传则不渲染）✅
- StoreEditor、BrandEditor、ProductEditor detail 的 `setLastSavedAt` 接入正确 ✅
- AgronomistEditor draft 模式 `handleDraftSave` + `handleDetailAdminSave` 均有 `setLastSavedAt` ✅
- `dayjs().locale(locale).fromNow()` 实例级 locale 设置正确，不影响全局 ✅
- 每分钟 `setInterval` 有在 `lastSavedAt` 变化时正确清理（useEffect cleanup）✅
- ProductEditor detail 模式 `form.submit()` → `handleSaveProduct` → `setLastSavedAt` 链路正确 ✅
- `BrandEditor.handleDetailSaveDraft` 成功后立即跳走，不停留页面，`setLastSavedAt` 缺失不影响实际体验 ✅（仅一致性建议）
