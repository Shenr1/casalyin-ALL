# task-field-scan-ingredientes

> 化学成分（ingredientes）模块字段对齐扫描（只扫描，不改代码）

## 扫描范围

| # | API | 方法 | 代理路由 | 后端路径 |
|---|-----|------|---------|---------|
| 1 | composition/query | POST | app/api/composition/query/route.ts | /api/composition/public/query |
| 2 | composition/listAll | GET | app/api/composition/listAll/route.ts | /api/composition/public/listAll |
| 3 | composition/by-slug/{slug} | GET | app/api/composition/by-slug/[slug]/route.ts | /api/composition/by-slug/{slug}（无 /public/） |

## 前端类型定义

### CompositionListVO (lib/list-api.ts) — 列表页+筛选器
- compositionId: number
- compositionName: string
- compositionCode?: string
- description?: string
- disabledFlag?: boolean
- updateTime?: string
- createTime?: string

### CompositionQueryForm (lib/list-api.ts)
- pageNum: number
- pageSize: number
- keyword?: string

## 状态: completed
