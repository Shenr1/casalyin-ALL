# task-field-scan-empresas

> 品牌模块字段对齐扫描（只扫描，不改代码）

## 扫描范围

| # | API | 方法 | 代理路由 | 后端路径 |
|---|-----|------|---------|---------|
| 1 | brand/query | POST | app/api/brand/query/route.ts | /api/brand/public/query |
| 2 | brand/detail/{id} | GET | app/api/brand/detail/[brandId]/route.ts | /api/brand/public/detail/{id} |
| 3 | brand/listAll | GET | app/api/brand/listAll/route.ts | /api/brand/public/listAll |
| 4 | brand/public/by-slug/{slug} | GET | **无代理路由** | /api/brand/public/by-slug/{slug} |

## 前端类型定义

### BrandListVO (lib/list-api.ts)
- brandId: number
- brandName: string
- brandCode: string
- description?: string
- disabledFlag?: boolean
- updateTime?: string
- createTime?: string
- vipLevel?: number
- vipLevelName?: string

### BrandDetailVO (lib/brand-detail-api.ts)
- brandId: number
- brandName: string
- brandCode: string
- logoUrl?: string
- bannerUrl?: string
- description: string
- disabledFlag: boolean
- updateTime: string
- createTime: string

### BrandQueryForm (lib/list-api.ts)
- pageNum: number
- pageSize: number
- keyword?: string

## 步骤

- [x] 读取列表页 app/empresas/page.tsx — 字段使用
- [x] 读取详情页 app/empresas/[slug]/page.tsx — 字段使用
- [x] 读取 lib/brand-detail-api.ts — 类型定义
- [x] 读取 lib/list-api.ts — BrandListVO + queryBrands + getAllBrands
- [x] 读取代理路由 app/api/brand/*/route.ts — 后端路径
- [x] 检查 by-slug 代理路由是否存在
- [x] 检查硬编码中文
- [x] 汇总报告

## 状态: completed
