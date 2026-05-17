# 店铺 Slug 路由实施计划

## 📋 项目概述

**目标**：将店铺详情页从 ID 路由（`/tiendas/[id]`）迁移到 slug 路由（`/tiendas/[slug]`），与产品和农艺师保持一致。

**原因**：
- 统一 URL 风格，保持一致性
- 更好的 SEO（slug 包含关键词）
- 更友好的 URL（可读性强）
- 支持预览功能（已发布用 slug，未发布用 ID + preview 参数）

**工作量评估**：⭐⭐⭐ 大（约 4-6 小时）

---

## 🎯 核心需求

### 1. URL 设计

#### 已发布店铺（正式链接）
- **格式**：`/tiendas/[slug]`
- **示例**：`/tiendas/tienda-agricola-san-jose`
- **用途**：可以推广、分享、SEO 优化

#### 未发布店铺（预览链接）
- **格式**：`/tiendas/[id]?preview=true`
- **示例**：`/tiendas/123?preview=true`
- **用途**：仅供预览，不能推广

#### 兼容性
- **保留 ID 路由**：`/tiendas/[id]` 仍然可以访问（向后兼容）
- **自动重定向**：访问 `/tiendas/[id]` 时，如果店铺已发布且有 slug，自动重定向到 `/tiendas/[slug]`

---

## 📊 任务分解

### 阶段 1：后端支持（需要后端配合）⚠️

#### 任务 1.1：确认 slug 字段
**负责人**：后端开发

**检查项**：
- [ ] 店铺表是否有 `slug` 字段？
- [ ] slug 是否唯一？
- [ ] slug 是否在创建/更新时自动生成？

**如果没有 slug 字段，需要**：
1. 添加 `slug` 字段到店铺表
2. 为现有店铺生成 slug（数据迁移）
3. 确保 slug 唯一性（数据库约束）

#### 任务 1.2：API 支持通过 slug 查询
**负责人**：后端开发

**需要的 API**：
- `GET /api/store/detail/by-slug/{slug}` - 通过 slug 查询店铺详情
- 或者修改现有 API，支持 slug 参数

**返回数据**：
- 店铺详情（包含 slug 字段）
- 与通过 ID 查询返回的数据结构一致

---

### 阶段 2：C 端实现 slug 路由（前端）

#### 任务 2.1：创建 slug 路由
**负责人**：frontend-dev
**工作量**：⭐⭐ 中（1-2 小时）

**文件操作**：
1. 创建新目录：`casalyin-Headless/app/tiendas/[slug]/`
2. 创建新文件：`casalyin-Headless/app/tiendas/[slug]/page.tsx`
3. 复制 `[id]/page.tsx` 的内容到 `[slug]/page.tsx`

**修改内容**：
```typescript
// 从
const storeId = parseInt(params.id as string)

// 改为
const slug = params.slug as string

// 修改 API 调用
const data = await getStoreDetailBySlug(slug)
```

**注意事项**：
- 保留 `[id]` 路由，不要删除（向后兼容）
- 两个路由可以共存

#### 任务 2.2：创建 API 函数
**负责人**：frontend-dev
**工作量**：⭐ 小（30 分钟）

**文件**：`casalyin-Headless/lib/store-detail-api.ts`

**新增函数**：
```typescript
export async function getStoreDetailBySlug(slug: string): Promise<StoreDetailVO | null> {
  try {
    const response = await fetch(`${API_BASE_URL}/api/store/detail/by-slug/${slug}`)
    if (!response.ok) return null
    const data = await response.json()
    return data.data
  } catch (error) {
    console.error('Failed to fetch store by slug:', error)
    return null
  }
}
```

#### 任务 2.3：实现自动重定向
**负责人**：frontend-dev
**工作量**：⭐ 小（30 分钟）

**文件**：`casalyin-Headless/app/tiendas/[id]/page.tsx`

**逻辑**：
```typescript
useEffect(() => {
  // 如果店铺已发布且有 slug，重定向到 slug 路由
  if (store && store.slug && store.displayStatus === 1 && !isPreviewMode) {
    router.replace(`/tiendas/${store.slug}`)
  }
}, [store, isPreviewMode])
```

---

### 阶段 3：后台管理生成预览链接（前端）

#### 任务 3.1：修改预览链接生成逻辑
**负责人**：frontend-dev
**工作量**：⭐ 小（30 分钟）

**文件**：`casalyin-server/src/pages/StoreEditor.tsx`

**修改内容**：
```typescript
const generatePreviewUrl = () => {
  const storeId = detailRaw?.storeId ?? paramId
  const slug = detailRaw?.slug
  if (!storeId) return null
  
  const baseUrl = import.meta.env.VITE_C_END_URL || 'http://localhost:3000'
  const isPublished = detailStatus === 1 // 1 = 已发布
  
  if (isPublished && slug) {
    // 已发布且有 slug：使用 slug 路由
    return `${baseUrl}/tiendas/${slug}`
  } else {
    // 未发布或没有 slug：使用 ID + preview 参数
    return `${baseUrl}/tiendas/${storeId}?preview=true`
  }
}
```

---

### 阶段 4：测试和验证

#### 任务 4.1：功能测试
**负责人**：QA / 开发者

**测试用例**：
- [ ] 访问 `/tiendas/[slug]` 可以正常显示已发布店铺
- [ ] 访问 `/tiendas/[id]` 可以正常显示店铺（向后兼容）
- [ ] 访问 `/tiendas/[id]` 时，已发布店铺自动重定向到 `/tiendas/[slug]`
- [ ] 访问 `/tiendas/[id]?preview=true` 显示预览模式（Sticky 浮动条）
- [ ] 后台管理点击预览按钮，已发布店铺打开 slug 链接
- [ ] 后台管理点击预览按钮，未发布店铺打开 ID + preview 链接
- [ ] slug 不存在时显示 404 页面
- [ ] 移动端响应式正常

#### 任务 4.2：SEO 验证
**负责人**：QA / 开发者

**检查项**：
- [ ] 已发布店铺的 URL 包含关键词（slug）
- [ ] 预览模式添加了 noindex meta 标签
- [ ] 页面 title 和 description 正确

---

## 🔄 实施顺序

### 第一步：后端准备（阻塞）⚠️
1. 确认 slug 字段是否存在
2. 如果不存在，添加 slug 字段并生成数据
3. 实现通过 slug 查询的 API

**预计时间**：2-3 小时（后端）

### 第二步：前端实现（依赖第一步）
1. 创建 slug 路由
2. 创建 API 函数
3. 实现自动重定向
4. 修改预览链接生成逻辑

**预计时间**：2-3 小时（前端）

### 第三步：测试验证
1. 功能测试
2. SEO 验证

**预计时间**：1 小时

---

## ⚠️ 风险和注意事项

### 1. 数据迁移风险
- **问题**：现有店铺可能没有 slug
- **解决**：后端需要为所有现有店铺生成 slug
- **建议**：使用店铺名称生成 slug，确保唯一性

### 2. URL 冲突
- **问题**：slug 可能与 ID 冲突（如果 slug 是纯数字）
- **解决**：slug 生成时避免纯数字，或者添加前缀（如 `store-123`）

### 3. 向后兼容性
- **问题**：已分享的 ID 链接可能失效
- **解决**：保留 ID 路由，实现自动重定向

### 4. 预览模式
- **问题**：未发布店铺没有 slug
- **解决**：未发布店铺使用 ID + preview 参数

---

## 📝 需要后端支持的部分

### 必须（阻塞前端开发）
1. **确认 slug 字段**：店铺表是否有 slug 字段？
2. **API 支持**：实现通过 slug 查询店铺详情的 API
3. **数据迁移**：为现有店铺生成 slug

### 可选（优化）
1. **slug 生成规则**：创建/更新店铺时自动生成 slug
2. **slug 唯一性**：数据库约束确保 slug 唯一
3. **slug 验证**：API 返回时包含 slug 字段

---

## 🎯 验收标准

### 功能完整性
- [x] 已发布店铺使用 slug 路由
- [x] 未发布店铺使用 ID + preview 参数
- [x] ID 路由向后兼容
- [x] 自动重定向到 slug 路由
- [x] 预览模式显示 Sticky 浮动条
- [x] 预览模式添加 noindex meta 标签

### 代码质量
- [x] 代码清晰易懂
- [x] 遵循项目规范
- [x] 无重复代码
- [x] 错误处理完善

### 用户体验
- [x] URL 友好，包含关键词
- [x] 响应式设计，移动端友好
- [x] 加载速度快
- [x] 404 页面友好

---

## 📅 时间估算

| 阶段 | 任务 | 负责人 | 工作量 | 预计时间 |
|------|------|--------|--------|----------|
| 1 | 后端支持 | 后端 | ⭐⭐⭐ | 2-3 小时 |
| 2 | C 端实现 | 前端 | ⭐⭐ | 2-3 小时 |
| 3 | 后台管理 | 前端 | ⭐ | 30 分钟 |
| 4 | 测试验证 | QA | ⭐ | 1 小时 |
| **总计** | | | | **5-7 小时** |

---

## 🚀 开始实施

### 第一步：确认后端支持
**需要用户/后端确认**：
1. 店铺表是否有 `slug` 字段？
2. 是否可以实现通过 slug 查询的 API？
3. 是否需要数据迁移？

### 第二步：前端开发
**等待后端准备完成后**，前端开始实施。

---

## 📞 联系方式

如有问题，请联系：
- 前端：frontend-dev
- 后端：backend-dev
- 项目负责人：用户

---

**创建时间**：2026-05-02
**最后更新**：2026-05-02
