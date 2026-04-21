# task-hydration-fix - 任务计划

> 状态: completed
> 分配: frontend-dev
> 创建: 2026-04-19

## 目标

修复首页 NotificationBar hydration 报错：服务端渲染西班牙语、客户端渲染中文。

## 根因

`lib/i18n/index.ts` 的 `getLocale()` 在 `NEXT_PUBLIC_LOCALE` 未被读取到时，回退到用 `NODE_ENV` 判断语言（production→es, 其他→zh）。Server SSR 和 Client hydration 对 `NODE_ENV` 的解析可能不一致，导致 locale 不匹配。

## 修复

将 `getLocale()` 简化为仅依赖 `NEXT_PUBLIC_LOCALE`，在模块加载时一次性解析并缓存，去除 `NODE_ENV` 回退逻辑。默认 `es`（生产默认语言）。

## 变更文件

- `casalyin-Headless/lib/i18n/index.ts` — 简化 `getLocale()`
