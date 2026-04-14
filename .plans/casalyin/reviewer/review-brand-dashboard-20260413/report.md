# 代码审查报告 — 品牌管理页面重构

**日期**：2026-04-13  
**审查人**：reviewer  
**请求方**：frontend-dev  
**涉及文件**：
- src/config/routes.tsx
- src/api/brand.ts（新增 BrandProductItem + apiGetBrandProducts）
- src/pages/BrandDashboard.tsx
- src/locales/zh-CN.json / en.json / es-ES.json

## 最终结论

[WARN] — 无 CRITICAL / HIGH 问题；存在 3 处 MEDIUM + 1 处 LOW，需 frontend-dev 知悉并择机修复。

## 问题清单

### MEDIUM-1：apiGetBrandProducts 接口路径为占位状态
brand.ts L248-249，注释写明待确认但已被实际调用，catch 静默吞掉错误，导致功能静默失效。

### MEDIUM-2：产品加载失败完全静默
BrandDashboard.tsx L63，违反"静默失败是 Bug"约定，建议 catch 里给 Table 设置 emptyText 或小型 Alert。

### MEDIUM-3：路由重复 path: '/brands'
routes.tsx L45 & L47，ADMIN 父节点与非 ADMIN 扁平入口同路径，建议 ADMIN 父节点改为虚拟路径。

### LOW-1：硬编码兜底 '刷新'
BrandDashboard.tsx L92，t('common.refresh') || '刷新' 违反禁止硬编码规范，需检查 key 是否存在后补全或删除兜底。

## 通过项
- 无 Modal.confirm / message.success 静态方法 ✅
- 无独立 XxxCreate/XxxDetail 页面 ✅
- brandDashboard.productList.* 三语言同步(7 key) ✅
- brandDashboard.trafficData.* 三语言同步(2 key) ✅
- menu.brand/brandList/brandApply 三语言同步 ✅
- Table 无操作列，点击行跳转 /products/:id ✅
