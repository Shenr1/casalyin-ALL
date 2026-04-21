# task-field-scan-plagas

> 病虫害模块字段对齐扫描（只扫描，不改代码）

## 扫描范围

| # | API | 方法 | 代理路由 | 后端路径 |
|---|-----|------|---------|---------|
| 1 | pest/query | POST | app/api/pest/query/route.ts | /api/pest/public/query |
| 2 | pest/listAll | GET | app/api/pest/listAll/route.ts | /api/pest/public/listAll |
| 3 | pest/by-slug/{slug} | GET | app/api/pest/by-slug/[slug]/route.ts | /api/pest/by-slug/{slug}（无 /public/） |
| 4 | pest/agronomists/{pestId} | GET | app/api/pest/agronomists/[pestId]/route.ts | /api/agronomist/public/pest/{pestId}/agronomists |

## 前端类型定义

### PestListVO (lib/list-api.ts) — 列表页
- pestId: number
- pestName: string
- description?: string

### PestQueryForm (lib/list-api.ts)
- pageNum: number
- pageSize: number
- keyword?: string

## 状态: completed
