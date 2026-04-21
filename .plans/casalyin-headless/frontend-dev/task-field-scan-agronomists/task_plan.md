# task-field-scan-agronomists

> 农艺师模块字段对齐扫描（只扫描，不改代码）

## 扫描范围

| # | API | 方法 | 代理路由 | 后端路径 |
|---|-----|------|---------|---------|
| 1 | agronomist/list | GET | app/api/agronomist/list/route.ts | /api/agronomist/public/list |
| 2 | agronomist/detail/{slug} | GET | app/api/agronomist/detail/[slug]/route.ts | /api/agronomist/public/detail/{slug} |
| 3 | crop/agronomists/{cropId} | GET | app/api/crop/agronomists/[cropId]/route.ts | /api/agronomist/public/crop/{cropId}/agronomists |
| 4 | pest/agronomists/{pestId} | GET | app/api/pest/agronomists/[pestId]/route.ts | /api/agronomist/public/pest/{pestId}/agronomists |
| 5 | agronomists (filter) | GET | app/api/agronomists/route.ts | /api/agronomist/public/list/filter |
| 6 | agronomists/{id} (by ID) | GET | app/api/agronomists/[id]/route.ts | /api/agronomist/public/detail/id/{id} |

## 前端类型定义

### AgronomistPublicItem — 关联专家卡片
- id, name?, title?, avatarUrl?, slug?

### AgronomistListItem — 列表页
- id, name?, title?, organization?, avatarUrl?, slug

### AgronomistDetailItem — 详情页
- id, name?, title?, organization?, avatarUrl?, profile?, featuredImage?, featuredContent?, cropIds?, cropNames?, pestIds?, pestNames?

## 状态: completed
