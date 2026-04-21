# task-field-scan-cultivos

> 作物模块字段对齐扫描（只扫描，不改代码）

## 扫描范围

| # | API | 方法 | 代理路由 | 后端路径 |
|---|-----|------|---------|---------|
| 1 | crop/listAll | GET | app/api/crop/listAll/route.ts | /api/crop/public/listAll |
| 2 | crop/by-slug/{slug} | GET | app/api/crop/by-slug/[slug]/route.ts | /api/crop/by-slug/{slug}（注意：无 /public/） |
| 3 | crop/agronomists/{cropId} | GET | app/api/crop/agronomists/[cropId]/route.ts | /api/agronomist/public/crop/{cropId}/agronomists |

## 前端类型定义

### CropListVO (lib/list-api.ts) — 列表页
- cropId: number
- cropName: string
- description?: string

### CropItem (lib/crop-pest-api.ts) — 死代码，未被任何文件 import
- id: number
- cropName: string

## 状态: completed
