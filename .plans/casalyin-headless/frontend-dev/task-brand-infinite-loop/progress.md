# task-brand-infinite-loop - 进度日志

## 2026-04-19

- [x] 定位问题页面: `app/empresas/page.tsx`
- [x] 找到根因: `useTranslation()` 每次渲染返回新 `t` 闭包，被用作 `useEffect` 依赖导致无限循环
- [x] 确认影响范围: empresas/ingredientes/plagas 三个列表页 + Breadcrumb 组件
- [x] 修复 `lib/i18n/client.ts`: 用 `useCallback` 包裹 `t` 函数
- [x] TypeScript 编译通过
- [x] 完成
