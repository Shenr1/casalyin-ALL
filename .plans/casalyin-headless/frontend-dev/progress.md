# frontend-dev - 工作日志

> 用于上下文恢复。压缩/重启后先读此文件。

---

## 2026-04-19

### task-hydration-fix (completed)
- 修复首页 NotificationBar hydration 报错
- 根因: `lib/i18n/index.ts` 的 `getLocale()` 用 `NODE_ENV` 做 locale 回退，server/client 解析不一致
- 修复: 去除 `NODE_ENV` 回退，改为模块顶层常量 `resolvedLocale`，仅依赖 `NEXT_PUBLIC_LOCALE`
- TypeScript 编译通过

### task-brand-infinite-loop (completed)
- 修复 `/empresas` 等列表页 `/api/brand/query` 无限调用
- 根因: `useTranslation()` 每次渲染返回新 `t` 闭包 → `useEffect` 依赖 `t` → 无限循环
- 影响: empresas/ingredientes/plagas 三个列表页 + Breadcrumb
- 修复: `lib/i18n/client.ts` 用 `useCallback` 包裹 `t`，稳定引用
- TypeScript 编译通过

### Phase 1 fixes (completed)

**Fix 1：品牌 by-slug 代理路由创建**
- 新建 `app/api/brand/public/by-slug/[slug]/route.ts`
- 转发到后端 `/api/brand/public/by-slug/{slug}`
- 修复品牌详情页通过 slug 直接访问时 404 的问题

**Fix 2：crop/pest/composition by-slug 后端路径验证**
- 测试 `http://localhost:8690/api/crop/by-slug/test-slug` → 200
- 测试 `http://localhost:8690/api/pest/by-slug/test-slug` → 200
- 测试 `http://localhost:8690/api/composition/by-slug/test-slug` → 200
- 结论：后端不要求 `/public/` 段，当前路径功能正常，标记为 LOW（命名不一致但不影响功能）

**Fix 3：plagas 详情页农艺师卡片改为可点击**
- 修改 `app/plagas/[slug]/page.tsx`
- 添加 `Link` import，将 `<div>` 替换为 `<Link href={...}>`
- 对齐 cultivos/productos 详情页的同类组件写法
- TypeScript 编译通过

### Phase 2 fixes (completed)

**Fix 4：ProductBlogLinks 接真实数据**
- 重写 `components/product-detail/ProductBlogLinks.tsx`
- 移除全部 mock 数据逻辑（getMockBlogArticles、formatUpdateTime、BlogArticle 接口）
- 改为解析 `blogUrls`（支持 `string[]` 和 `string` 两种类型）
- 空数组时 `return null`，不渲染区块
- 最多展示 3 条链接，保留外部链接图标和 hover 效果

**Fix 5：品牌列表卡片补 logoUrl**
- `lib/list-api.ts` 的 `BrandListVO` 补上 `logoUrl?: string`
- `app/empresas/page.tsx` 添加 `Image` import
- 卡片区域：有 `logoUrl` 时渲染 `<Image>`（48x48, object-contain），无则保留 emoji 占位

**Fix 6：农艺师详情页渲染 featuredImage / featuredContent**
- `app/agronomists/[slug]/page.tsx` 在 profile 区块后添加两个条件渲染区块
- `featuredImage`：`<Image>` 全宽圆角，800x400，object-cover
- `featuredContent`：`whitespace-pre-wrap` 文本块
- 两个字段都是有值才显示

**Fix 7：empresas 图片 alt 修复**
- `app/empresas/[slug]/page.tsx` banner `alt=""` → `alt={brand.brandName + ' banner'}`
- logo `alt=""` → `alt={brand.brandName + ' logo'}`
- TypeScript 编译通过

### Phase 3 — i18n 硬编码清理 (completed)

**新增 i18n key（两个语言文件同步）：**
- `common.download` — 下载 / Descargar
- `common.notFound` — 未找到 / No encontrado
- `pages.product.documents` — 产品文档 / Documentos

**empresas 详情页** — 12处全部替换为 t() 调用
**productos 详情页** — 12处全部替换为 t() 调用
**cultivos 列表页** — 4处全部替换，添加 useTranslation
**agronomists 列表页** — 1处替换

### Phase 4 — 代码卫生 (completed)

- 删除 `lib/crop-pest-api.ts`（死代码，零 import）
- 删除 `app/api/agronomists/` 目录（冗余复数路由，零 import）
- TypeScript 编译零错误
