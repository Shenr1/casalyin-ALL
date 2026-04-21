# task-hydration-fix - 进度日志

## 2026-04-19

- [x] 定位报错组件: Header.tsx → NotificationBar.tsx，翻译 key 为 `notification.message1`
- [x] 分析根因: `getLocale()` 的 `NODE_ENV` 回退导致 server/client locale 不一致
- [x] 修复 `lib/i18n/index.ts`: 去除 `NODE_ENV` 回退，改为模块顶层常量
- [x] TypeScript 编译通过（`tsc --noEmit` 无错误）
- [x] 完成
