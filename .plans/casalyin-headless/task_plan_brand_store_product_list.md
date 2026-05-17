# 品牌/店铺详情页产品列表改造

## 需求来源
用户口述（2026-05-10），上下文压缩前的讨论内容。

## 改造范围
只改两个页面：
- /empresas/[slug]（品牌详情页）
- /tiendas/[slug]（店铺详情页）

成分、农作物、虫害详情页不动。

## 布局变化
- 现在：左侧过滤器 + 右侧产品列表（一行一个）
- 目标：无左侧过滤器，产品通屏，桌面3列 / 平板2列 / 移动1列

## 顶部筛选条（两个页面共有）
- isRequired: true 的标签分类 → 显示为 tab 或 chip 筛选
- isRequired: false 的标签分类 → 不显示
- 新增名称搜索框

## 品牌详情页额外：地区下拉菜单
- 过滤的是产品的 availableCities（可售城市）
- 数据来源：动态聚合该品牌下所有产品的 availableCities，去重后作为下拉选项
- 店铺详情页不需要地区过滤

## 地区过滤技术方案

### 现状
- 产品已有 availableCities 字段（JSON 字符串，存城市名数组）
- 后台 ProductEditor 已支持设置可售城市（PlacesCityAutocomplete）
- C端产品详情页已展示 availableCities
- 缺：按城市过滤产品列表的能力

### 需要新增

**后端（casalyin-java）：**
1. ProductFilterForm 加 `city` 字段（String）
2. ProductMapper.xml 加 city 过滤条件（JSON_CONTAINS 或 LIKE '%"城市名"%'）
3. 新增接口：GET /product/public/available-cities?brandId=xxx
   - 聚合该品牌下所有产品的 available_cities，返回去重城市列表

**C端 Next.js API 代理（casalyin-Headless）：**
4. 新增 /api/product/available-cities/route.ts（代理后端接口）

**C端页面（casalyin-Headless）：**
5. 品牌详情页加地区下拉，选中后加入 filterForm.city

## 店铺详情页产品列表筛选
店铺产品走 /api/store/public/query-products，需确认后端是否支持 keyword 和 tagIds 过滤。
如不支持，需同步扩展后端 StoreProductQueryForm。

## 进度
- [ ] 后端：ProductFilterForm 加 city 字段
- [ ] 后端：ProductMapper.xml 加 city 过滤
- [ ] 后端：新增 available-cities 聚合接口
- [ ] C端：新增 /api/product/available-cities 代理
- [ ] C端：品牌详情页改造（布局 + 顶部筛选条 + 地区下拉）
- [ ] C端：店铺详情页改造（布局 + 顶部筛选条）
- [ ] 确认店铺产品接口是否支持 keyword/tagIds 过滤
