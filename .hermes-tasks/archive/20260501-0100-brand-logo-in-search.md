# 首页搜索品牌结果展示 Logo

**创建时间**: 2026-05-01 01:00  
**状态**: ✅ 已完成并合并到 main  
**负责**: Hermes  
**分支**: feature/brand-logo-and-related-content（已合并删除）

## 需求
首页搜索品牌时，如果品牌有 logo，优先展示 logo；鼠标悬停时显示品牌名称。

## 实施内容

### 后端（casalyin-java）
**分支**: feature/brand-logo-in-search（已合并到 main）

1. 修改 `BrandSearchVO.java`，添加 `brandLogoUrl` 字段
2. 修改 `BrandMapper.xml`，在搜索查询中返回 `brand_logo_url`

### 前端（casalyin-Headless）
**分支**: feature/brand-logo-and-related-content（已合并到 main）

1. 修改 `lib/search-api.ts`
   - 在 `ApiBrandSearchVO` 接口添加 `brandLogoUrl?: string`
   - 在 `SearchResult` 接口添加 `logoUrl?: string`
   - 在品牌搜索结果映射中添加 `logoUrl: brand.brandLogoUrl`

2. 修改 `components/dimensions/BrandsResults.tsx`
   - 判断品牌是否有 logo
   - 有 logo：展示 logo 图片，hover 时显示品牌名称
   - 无 logo：保持原有展示方式（品牌名称）

## 技术细节

### UI 交互
- Logo 尺寸：80x80px，圆角 8px
- Hover 效果：显示品牌名称的 tooltip
- 降级处理：无 logo 时显示品牌名称

### 代码示例
```typescript
{brand.logoUrl ? (
  <div className="relative group">
    <img
      src={brand.logoUrl}
      alt={brand.name}
      className="w-20 h-20 object-contain rounded-lg"
    />
    <div className="absolute inset-0 bg-black/60 opacity-0 group-hover:opacity-100 
                    transition-opacity flex items-center justify-center rounded-lg">
      <span className="text-white text-sm font-medium px-2 text-center">
        {brand.name}
      </span>
    </div>
  </div>
) : (
  <span className="text-lg font-semibold">{brand.name}</span>
)}
```

## 验证
- ✅ 后端 API 返回 brandLogoUrl 字段
- ✅ 前端正确展示 logo
- ✅ Hover 交互正常
- ✅ 无 logo 时降级展示正常
- ✅ 代码已合并到 main 分支

## 相关提交
- casalyin-java: commit a1bff2a
- casalyin-Headless: commit 572aad1
