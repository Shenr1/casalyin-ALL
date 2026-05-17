# 店铺 Banner 功能实施计划

## 📋 项目概述

**目标**：给店铺添加 banner 字段，和品牌一样的逻辑。

**参考**：品牌的 bannerUrl 字段实现。

**工作量评估**：⭐⭐ 中（约 2-3 小时）

---

## 🎯 核心需求

### Banner 字段
- **字段名称**：`bannerUrl`
- **字段类型**：VARCHAR(500)
- **用途**：店铺详情页顶部 banner 图片

### 功能要求
- 后台管理可以上传/编辑 banner
- C 端店铺详情页显示 banner
- 与品牌的 banner 逻辑一致

---

## 📊 任务分解

### 阶段 1：后端实现

#### 任务 1.1：数据库迁移
**负责人**：backend-dev
**工作量**：⭐ 小（15 分钟）

**文件**：`casalyin-java/casalyin-admin/src/main/resources/db/migration/V27__add_store_banner.sql`

**内容**：
```sql
-- 添加 bannerUrl 字段
ALTER TABLE t_store ADD COLUMN banner_url VARCHAR(500) COMMENT '店铺 banner 图片 URL';
```

#### 任务 1.2：修改实体类和 VO
**负责人**：backend-dev
**工作量**：⭐ 小（15 分钟）

**文件**：
1. `StoreEntity.java` - 添加 `bannerUrl` 字段
2. `StoreVO.java` - 添加 `bannerUrl` 字段
3. `StoreDraftEntity.java` - 添加 `bannerUrl` 字段
4. `StoreDraftVO.java` - 添加 `bannerUrl` 字段

**代码**：
```java
/**
 * 店铺 banner 图片 URL
 */
private String bannerUrl;
```

#### 任务 1.3：修改表单
**负责人**：backend-dev
**工作量**：⭐ 小（15 分钟）

**文件**：
1. `StoreAddForm.java` - 添加 `bannerUrl` 字段
2. `StoreUpdateForm.java` - 添加 `bannerUrl` 字段
3. `StoreDraftUpdateForm.java` - 添加 `bannerUrl` 字段

#### 任务 1.4：修改 Mapper
**负责人**：backend-dev
**工作量**：⭐ 小（15 分钟）

**文件**：`StoreMapper.xml`

**修改**：在所有查询语句中添加 `banner_url` 字段。

#### 任务 1.5：修改 Service（草稿对比）
**负责人**：backend-dev
**工作量**：⭐ 小（15 分钟）

**文件**：`StoreDraftService.java`

**修改**：在草稿对比逻辑中添加 `bannerUrl` 字段的对比。

参考品牌的实现：
```java
addDiffItem(diff, "bannerUrl", "Banner", old.getBannerUrl(), draft.getBannerUrl());
```

---

### 阶段 2：前端后台实现

#### 任务 2.1：添加 banner 上传功能
**负责人**：frontend-dev
**工作量**：⭐⭐ 中（30 分钟）

**文件**：`casalyin-server/src/pages/StoreEditor.tsx`

**参考**：品牌编辑器（`BrandEditor.tsx`）的 banner 上传功能。

**添加内容**：
1. 在表单中添加 banner 上传字段
2. 使用 `ImageUpload` 组件
3. 表单提交时包含 `bannerUrl`

**位置**：在店铺名称下方，店铺信息上方。

---

### 阶段 3：前端 C 端实现

#### 任务 3.1：显示 banner
**负责人**：frontend-dev
**工作量**：⭐⭐ 中（30 分钟）

**文件**：`casalyin-Headless/app/tiendas/[slug]/page.tsx`

**参考**：品牌详情页的 banner 显示。

**添加内容**：
```tsx
{/* Banner */}
{store.bannerUrl && (
  <div className="w-full h-64 md:h-96 relative mb-8">
    <img
      src={store.bannerUrl}
      alt={store.storeName}
      className="w-full h-full object-cover"
    />
  </div>
)}
```

**位置**：在页面顶部，店铺名称上方。

---

## 🔄 实施顺序

### 第一步：后端实现
1. 数据库迁移
2. 修改实体类和 VO
3. 修改表单
4. 修改 Mapper
5. 修改 Service

**预计时间**：1 小时

### 第二步：前端后台实现
1. 添加 banner 上传功能

**预计时间**：30 分钟

### 第三步：前端 C 端实现
1. 显示 banner

**预计时间**：30 分钟

### 第四步：测试验证
1. 后台管理上传 banner
2. C 端查看 banner 显示

**预计时间**：30 分钟

---

## ⚠️ 注意事项

### 1. 字段命名
- 数据库：`banner_url`（下划线）
- Java：`bannerUrl`（驼峰）
- 保持与品牌一致

### 2. 图片尺寸
- 建议尺寸：1920x400 或类似宽屏比例
- 文件大小限制：参考现有图片上传限制

### 3. 响应式设计
- 移动端：高度适配（如 h-64）
- 桌面端：高度适配（如 md:h-96）

### 4. 默认值
- 如果没有 banner，不显示 banner 区域
- 不需要默认占位图

---

## 📝 验收标准

### 后端
- [ ] 数据库添加了 `banner_url` 字段
- [ ] 实体类和 VO 添加了 `bannerUrl` 字段
- [ ] 表单添加了 `bannerUrl` 字段
- [ ] Mapper 查询包含 `banner_url` 字段
- [ ] 草稿对比包含 `bannerUrl` 字段

### 前端后台
- [ ] 可以上传 banner 图片
- [ ] 可以预览 banner 图片
- [ ] 可以删除 banner 图片
- [ ] 保存后 banner 正确存储

### 前端 C 端
- [ ] 有 banner 时正确显示
- [ ] 没有 banner 时不显示 banner 区域
- [ ] 响应式设计正常
- [ ] 图片加载正常

---

## 📅 时间估算

| 阶段 | 任务 | 负责人 | 工作量 | 预计时间 |
|------|------|--------|--------|----------|
| 1 | 后端实现 | backend-dev | ⭐⭐ | 1 小时 |
| 2 | 前端后台 | frontend-dev | ⭐ | 30 分钟 |
| 3 | 前端 C 端 | frontend-dev | ⭐ | 30 分钟 |
| 4 | 测试验证 | QA | ⭐ | 30 分钟 |
| **总计** | | | | **2.5 小时** |

---

**创建时间**：2026-05-02
**最后更新**：2026-05-02
