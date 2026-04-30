# 品牌详情页关联内容 API 需求

## 背景
品牌详情页需要展示该品牌创建的农艺师和店铺，参考产品详情页的 UI 设计。

## 需求

### 1. 获取品牌关联的农艺师列表
**接口路径**: `GET /api/brand/agronomists/{brandId}`

**请求参数**:
- `brandId` (path): 品牌 ID

**响应格式**:
```json
{
  "code": 0,
  "ok": true,
  "msg": "success",
  "data": [
    {
      "id": 1,
      "name": "张三",
      "title": "高级农艺师",
      "slug": "zhang-san",
      "avatar": "https://...",
      "description": "专注于...",
      "crops": ["玉米", "小麦"]
    }
  ]
}
```

**业务逻辑**:
- 只返回该品牌创建的农艺师
- 只返回已发布/启用的农艺师
- 按创建时间倒序排列

### 2. 获取品牌关联的店铺列表
**接口路径**: `GET /api/brand/stores/{brandId}`

**请求参数**:
- `brandId` (path): 品牌 ID

**响应格式**:
```json
{
  "code": 0,
  "ok": true,
  "msg": "success",
  "data": [
    {
      "storeId": 1,
      "storeName": "绿色农资店",
      "address": "北京市朝阳区...",
      "phone": "010-12345678",
      "latitude": 39.9042,
      "longitude": 116.4074
    }
  ]
}
```

**业务逻辑**:
- 只返回该品牌创建的店铺
- 只返回已启用的店铺
- 按创建时间倒序排列

### 3. 品牌认领状态判断
**需求**: 只有品牌被认领后才展示农艺师和店铺

**实现方式**:
- 方案 1: 在品牌详情 API (`/api/brand/detail/{brandId}`) 中添加 `isClaimed` 字段
- 方案 2: 通过 `brandApply` 表判断是否有已通过的认领申请

**推荐方案 1**，修改 `BrandDetailVO` 添加字段：
```java
public class BrandDetailVO {
    // ... 现有字段
    private Boolean isClaimed; // 是否已被认领
    private Long claimedUserId; // 认领用户 ID（可选）
}
```

## 数据库表关联关系（需确认）

### 农艺师表
- 表名: `agronomist` 或 `t_agronomist`
- 关联字段: `brand_id` 或 `create_brand_id`

### 店铺表
- 表名: `store` 或 `t_store`
- 关联字段: `brand_id` 或 `create_brand_id`

### 品牌认领表
- 表名: `brand_apply` 或 `t_brand_apply`
- 关联字段: `brand_id`, `status` (审核状态)

## 实施步骤

### 后端（Java）
1. 确认数据库表结构和关联关系
2. 创建 VO 类：`AgronomistPublicVO`, `StorePublicVO`
3. 在 `BrandMapper.xml` 中添加查询 SQL
4. 在 `BrandService` 中添加业务方法
5. 在 `BrandController` 中添加接口
6. 修改 `BrandDetailVO` 添加 `isClaimed` 字段

### 前端（Next.js）

#### 1. 创建 API 调用函数
文件: `casalyin-Headless/lib/brand-detail-api.ts`

```typescript
// 添加到现有文件末尾

// 品牌关联的农艺师
export interface BrandAgronomistVO {
  id: number
  name: string
  title?: string
  slug: string
  avatar?: string
  description?: string
  crops?: string[]
}

export interface ApiBrandAgronomistsResponse {
  code: number
  level: string
  msg: string
  ok: boolean
  data: BrandAgronomistVO[]
  dataType: number
}

/**
 * 获取品牌关联的农艺师列表
 * @param brandId 品牌ID
 * @returns 农艺师列表
 */
export async function getBrandAgronomists(brandId: number): Promise<BrandAgronomistVO[]> {
  const apiUrl = `/api/brand/agronomists/${brandId}`

  try {
    const response = await fetch(apiUrl, {
      method: 'GET',
      headers: {
        'Accept': 'application/json',
      },
      cache: 'no-store'
    })

    if (!response.ok) {
      throw new Error(`请求失败: ${response.status}`)
    }

    const data: ApiBrandAgronomistsResponse = await response.json()

    if (!data.ok || data.code !== 0) {
      console.error(`获取品牌农艺师失败: ${data.msg}`)
      return []
    }

    return data.data || []
  } catch (error) {
    console.error(`获取品牌(${brandId})农艺师失败:`, error)
    return []
  }
}

// 品牌关联的店铺
export interface BrandStoreVO {
  storeId: number
  storeName: string
  address?: string
  phone?: string
  latitude?: number
  longitude?: number
}

export interface ApiBrandStoresResponse {
  code: number
  level: string
  msg: string
  ok: boolean
  data: BrandStoreVO[]
  dataType: number
}

/**
 * 获取品牌关联的店铺列表
 * @param brandId 品牌ID
 * @returns 店铺列表
 */
export async function getBrandStores(brandId: number): Promise<BrandStoreVO[]> {
  const apiUrl = `/api/brand/stores/${brandId}`

  try {
    const response = await fetch(apiUrl, {
      method: 'GET',
      headers: {
        'Accept': 'application/json',
      },
      cache: 'no-store'
    })

    if (!response.ok) {
      throw new Error(`请求失败: ${response.status}`)
    }

    const data: ApiBrandStoresResponse = await response.json()

    if (!data.ok || data.code !== 0) {
      console.error(`获取品牌店铺失败: ${data.msg}`)
      return []
    }

    return data.data || []
  } catch (error) {
    console.error(`获取品牌(${brandId})店铺失败:`, error)
    return []
  }
}
```

同时修改 `BrandDetailVO` 接口：
```typescript
export interface BrandDetailVO {
  brandId: number
  brandName: string
  brandCode: string
  logoUrl?: string
  bannerUrl?: string
  description: string
  disabledFlag: boolean
  updateTime: string
  createTime: string
  isClaimed?: boolean  // 新增：是否已被认领
}
```

#### 2. 修改品牌详情页
文件: `casalyin-Headless/app/empresas/[slug]/page.tsx`

在现有代码基础上添加：

**导入新的 API 函数**（第 5 行附近）：
```typescript
import { getBrandDetail, BrandDetailVO, getBrandAgronomists, getBrandStores, BrandAgronomistVO, BrandStoreVO } from '@/lib/brand-detail-api'
```

**添加状态管理**（第 52 行附近，`allStores` 下方）：
```typescript
const [brandAgronomists, setBrandAgronomists] = useState<BrandAgronomistVO[]>([])
const [brandStores, setBrandStores] = useState<BrandStoreVO[]>([])
```

**添加数据加载逻辑**（第 139 行附近，`fetchProducts` useEffect 后面）：
```typescript
// 3. 加载品牌关联的农艺师和店铺（仅当品牌已认领时）
useEffect(() => {
  const fetchBrandRelatedContent = async () => {
    if (!brand?.brandId || !brand?.isClaimed) return

    try {
      const [agronomists, stores] = await Promise.all([
        getBrandAgronomists(brand.brandId),
        getBrandStores(brand.brandId)
      ])
      
      setBrandAgronomists(agronomists)
      setBrandStores(stores)
    } catch (error) {
      console.error('加载品牌关联内容失败:', error)
    }
  }

  if (brand?.isClaimed) {
    fetchBrandRelatedContent()
  }
}, [brand?.brandId, brand?.isClaimed])
```

**修改店铺区域渲染**（第 294-295 行）：
```typescript
{/* 店铺区域 - 使用 ProductStores 组件 */}
{brand?.isClaimed && brandStores.length > 0 && (
  <ProductStores stores={brandStores.map(s => ({
    storeId: s.storeId,
    storeName: s.storeName,
    address: s.address || '',
    phone: s.phone || '',
    latitude: s.latitude,
    longitude: s.longitude
  }))} />
)}
```

**添加农艺师区域**（第 295 行后，店铺区域之前）：
```typescript
{/* 农艺师区域 */}
{brand?.isClaimed && brandAgronomists.length > 0 && (
  <div className="page-width-sm py-12 bg-white">
    <h2 className="text-2xl font-bold text-gray-900 mb-6">
      {t('pages.brand.relatedAgronomists')}
    </h2>
    <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
      {brandAgronomists.map((agronomist) => (
        <Link
          key={agronomist.id}
          href={`/agronomists/${agronomist.slug}`}
          className="bg-gray-50 rounded-lg p-6 hover:shadow-md transition-shadow border border-gray-200"
        >
          <div className="flex items-start gap-4">
            {agronomist.avatar ? (
              <img
                src={agronomist.avatar}
                alt={agronomist.name}
                className="w-16 h-16 rounded-full object-cover"
              />
            ) : (
              <div className="w-16 h-16 rounded-full bg-primary-100 flex items-center justify-center">
                <span className="text-2xl">👨‍🌾</span>
              </div>
            )}
            <div className="flex-1 min-w-0">
              <h3 className="font-semibold text-gray-900 mb-1">
                {agronomist.name}
              </h3>
              {agronomist.title && (
                <p className="text-sm text-gray-600 mb-2">{agronomist.title}</p>
              )}
              {agronomist.crops && agronomist.crops.length > 0 && (
                <div className="flex flex-wrap gap-1">
                  {agronomist.crops.slice(0, 3).map((crop, idx) => (
                    <span
                      key={idx}
                      className="text-xs bg-green-100 text-green-700 px-2 py-0.5 rounded"
                    >
                      {crop}
                    </span>
                  ))}
                </div>
              )}
            </div>
          </div>
        </Link>
      ))}
    </div>
  </div>
)}
```

#### 3. 添加国际化文案
文件: `casalyin-Headless/src/locales/zh-CN/translation.json`

在 `pages.brand` 下添加：
```json
{
  "pages": {
    "brand": {
      "relatedAgronomists": "品牌农艺师"
    }
  }
}
```

## 参考代码

### 产品详情页的店铺模块
文件: `casalyin-Headless/components/product-stores/ProductStores.tsx`

### 产品详情页的农艺师模块
文件: `casalyin-Headless/app/productos/[slug]/page.tsx` (第 137-150 行)

## 注意事项
1. 品牌未认领时，不展示农艺师和店铺模块
2. 农艺师和店铺列表为空时，不展示对应模块
3. 接口需要处理品牌不存在的情况（返回空数组）
4. 前端需要处理 loading 和 error 状态
