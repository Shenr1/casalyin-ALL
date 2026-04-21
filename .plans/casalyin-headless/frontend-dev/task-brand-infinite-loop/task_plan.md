# task-brand-infinite-loop - 任务计划

> 状态: completed
> 分配: frontend-dev
> 创建: 2026-04-19

## 目标

修复 `/empresas` 品牌列表页 `/api/brand/query` 无限调用。

## 根因

`useTranslation()` hook 每次渲染都创建新的 `t` 函数闭包。多个页面组件将 `t` 放入 `useEffect` 依赖数组 → 每次渲染触发 effect → fetch → setState → 重新渲染 → 无限循环。

## 影响范围

- `app/empresas/page.tsx` (L37)
- `app/ingredientes/page.tsx` (L36)
- `app/plagas/page.tsx` (L37)
- `components/Breadcrumb.tsx` (L79)

## 修复

`lib/i18n/client.ts`：用 `useCallback` 包裹返回的 `t` 函数，以 `locale` 为依赖（locale 是模块顶层常量，不会变化），保证引用稳定。

## 变更文件

- `casalyin-Headless/lib/i18n/client.ts`
