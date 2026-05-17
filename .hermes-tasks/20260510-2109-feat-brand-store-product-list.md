# 任务交接：品牌/店铺详情页产品列表改造

> 创建时间：2026-05-10 21:09
> 来源：.plans/casalyin-headless/task_plan_brand_store_product_list.md
> 状态：待开始（全部 TODO）

---

## 背景

改造两个 C 端详情页的产品列表区域：
- `/empresas/[slug]`（品牌详情页）
- `/tiendas/[slug]`（店铺详情页）

---

## 任务清单

### 【后端 backend-dev】casalyin-java

#### T1：ProductFilterForm 加 city 字段

文件：`casalyin-java/casalyin-admin/src/main/java/com/casalyin/admin/module/busniess/product/domain/form/ProductFilterForm.java`

在现有字段末尾加：

```java
@Schema(description = "可售城市过滤（精确匹配 available_cities JSON 数组中的城市名）")
private String city;
```

#### T2：ProductMapper.xml 加 city 过滤条件

文件：`casalyin-java/casalyin-admin/src/main/resources/mapper/product/ProductMapper.xml`

在 filterProduct 的动态 SQL 中加（available_cities 是 JSON 字符串列，存储格式为 `["城市A","城市B"]`）：

```xml
<if test="city != null and city != ''">
    AND JSON_CONTAINS(available_cities, CONCAT('"', #{city}, '"'))
</if>
```

#### T3：新增 available-cities 聚合接口

Controller：`ProductController.java`，在 `@NoNeedLogin` 区域新增：

```
GET /product/public/available-cities?brandId={brandId}
```

逻辑：查询该品牌下所有已发布产品的 available_cities 字段，解析 JSON 数组，去重后返回城市名列表。

返回格式：`ResponseDTO<List<String>>`

Service 方法建议放在 ProductService，Mapper 用原生 SQL：
```sql
SELECT available_cities FROM t_product
WHERE brand_id = #{brandId} AND status_flag = 1 AND disabled_flag = 0
AND available_cities IS NOT NULL AND available_cities != '[]'
```

#### T4：StoreProductQueryForm 加 keyword 和 tagIds

文件：`casalyin-java/casalyin-admin/src/main/java/com/casalyin/admin/module/busniess/store/domain/form/StoreProductQueryForm.java`

当前只有 storeId 和 publicOnly，需加：

```java
@Schema(description = "产品名称模糊搜索")
private String keyword;

@Schema(description = "标签ID列表过滤")
private List<Long> tagIds;
```

同步修改 StoreService 和 StoreMapper.xml 中 queryStoreProducts 的 SQL，加对应动态条件。

---

### 【C端 frontend-dev】casalyin-Headless

#### T5：新增 available-cities API 代理路由

新建文件：`casalyin-Headless/app/api/product/available-cities/route.ts`

代理后端 `GET /product/public/available-cities?brandId=xxx`，透传 brandId 参数。

#### T6：品牌详情页改造

文件：`casalyin-Headless/app/empresas/[slug]/page.tsx`

改造内容：
1. **布局**：去掉左侧 `<ProductFilter>` 组件，产品列表通屏展示
2. **产品网格**：桌面 3 列 / 平板 2 列 / 移动 1 列（Tailwind：`grid-cols-1 md:grid-cols-2 lg:grid-cols-3`）
3. **顶部筛选条**（新增，放在产品列表上方）：
   - 名称搜索框（输入后过滤 productName）
   - isRequired=true 的标签分类 → 显示为 chip/tab 筛选（调用现有 tagCategories 数据，过滤 isRequired=true 的分类）
   - 地区下拉菜单（调用 T5 的 available-cities 接口，选中后加入 filterForm.city）
4. 筛选条件变化时重新调用 `filterProducts`（已有的 `/api/product/filter` 代理）

注意：现有页面已有 `filterProducts` 和 `ProductFilterForm` 的调用逻辑，在此基础上扩展，不要重写整个页面。

#### T7：店铺详情页改造

文件：`casalyin-Headless/app/tiendas/[slug]/page.tsx`（或对应路径，请先确认实际文件位置）

改造内容：
1. **布局**：去掉左侧过滤器，产品列表通屏展示
2. **产品网格**：同上，3列/2列/1列
3. **顶部筛选条**（新增）：
   - 名称搜索框
   - isRequired=true 的标签分类 → chip/tab 筛选
   - 无地区下拉（店铺不需要）
4. 筛选条件变化时重新调用店铺产品接口（依赖 T4 后端扩展完成后才能接 keyword/tagIds）

---

## 依赖关系

```
T1 → T2（city 字段先加，再改 SQL）
T3 独立（可与 T1/T2 并行）
T4 独立（可与 T1/T2/T3 并行）
T5 依赖 T3（后端接口先完成）
T6 依赖 T1+T2+T3+T5（city 过滤 + available-cities 接口）
T7 依赖 T4（keyword/tagIds 过滤）
```

建议执行顺序：
1. 后端 T1+T2+T3+T4 并行开发
2. 后端完成后，C端 T5+T6+T7 并行开发

---

## 注意事项

1. **不要改动其他页面**：成分、农作物、虫害详情页不在本次范围
2. **Flyway**：本次改动不涉及 DB 结构变更（available_cities 字段已存在），无需新增迁移文件
3. **isRequired 数据来源**：TagCategory 的 isRequired 字段后端已打通（v2.0.1 已完成），前端直接用现有 tagCategories API 过滤即可
4. **发版前不要 push**：commit 后告知用户，由 Hermes 验证后 push
