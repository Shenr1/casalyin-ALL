# frontend-dev - 发现索引

> 纯索引——每个条目应简短（Status + Report 链接 + Summary）。

---

## task-hydration-fix
- Status: completed
- Report: [findings.md](task-hydration-fix/findings.md)
- Summary: 首页 hydration 报错，根因是 getLocale() 的 NODE_ENV 回退导致 server/client locale 不匹配，已修复为仅依赖 NEXT_PUBLIC_LOCALE

## task-brand-infinite-loop
- Status: completed
- Report: [task_plan.md](task-brand-infinite-loop/task_plan.md)
- Summary: useTranslation() 每次渲染返回新 t 闭包，被 useEffect 依赖导致 empresas/ingredientes/plagas 三个列表页无限 API 调用，已用 useCallback 稳定引用

## task-field-scan-productos
- Status: completed
- Report: [task_plan.md](task-field-scan-productos/task_plan.md)
- Summary: 产品模块字段对齐扫描，4个接口字段名全部 camelCase 一致，发现 prices 未使用、relatedFiles 未使用、部分硬编码中文文案

## task-field-scan-empresas
- Status: completed
- Report: [task_plan.md](task-field-scan-empresas/task_plan.md)
- Summary: 品牌模块字段扫描，3个API字段名camelCase一致；发现1个HIGH问题（by-slug缺代理路由）+BrandListVO缺logoUrl字段+详情页12处硬编码中文

## task-field-scan-agronomists
- Status: completed
- Report: [task_plan.md](task-field-scan-agronomists/task_plan.md)
- Summary: 农艺师模块字段扫描，6个API全有代理路由，字段名camelCase一致；发现2个MEDIUM问题（featuredImage/featuredContent未渲染、重复代理路由对）+列表页1处硬编码中文+plagas详情页农艺师卡片不可点击

## task-field-scan-cultivos
- Status: completed
- Report: [task_plan.md](task-field-scan-cultivos/task_plan.md)
- Summary: 作物模块字段扫描，3个API代理路由完整，字段camelCase一致；发现crop-pest-api.ts死代码+列表页无分页+3处硬编码中文+by-slug后端路径缺/public/+详情页无独立crop detail API

## task-field-scan-plagas
- Status: completed
- Report: [task_plan.md](task-field-scan-plagas/task_plan.md)
- Summary: 病虫害模块字段扫描，4个API代理路由完整，字段camelCase一致；与cultivos结构高度对称但i18n更完善；by-slug后端路径缺/public/+详情页农艺师卡片不可点击（同agronomists扫描#3）+无独立pest detail API

## task-field-scan-ingredientes
- Status: completed
- Report: [task_plan.md](task-field-scan-ingredientes/task_plan.md)
- Summary: 化学成分模块字段扫描，3个API代理路由完整，字段camelCase一致，i18n全覆盖无硬编码中文；by-slug后端路径缺/public/（跨模块共性）+无独立composition detail API+CompositionListVO有compositionCode/disabledFlag等冗余字段
