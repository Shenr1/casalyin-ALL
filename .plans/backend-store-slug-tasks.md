# 后端任务：店铺 Slug 支持

## 任务 1：添加 slug 字段

### 1.1 数据库迁移

```sql
-- 添加 slug 字段
ALTER TABLE t_store ADD COLUMN slug VARCHAR(255) UNIQUE COMMENT '店铺 slug（用于 URL）';

-- 为现有店铺生成 slug
UPDATE t_store 
SET slug = CONCAT('store-', store_id) 
WHERE slug IS NULL;

-- 添加唯一索引
CREATE UNIQUE INDEX idx_store_slug ON t_store(slug);
```

### 1.2 修改实体类

**文件**：`StoreEntity.java`

```java
/**
 * 店铺 slug（用于 URL）
 */
private String slug;
```

---

## 任务 2：实现通过 slug 查询的 API

### 2.1 Mapper 层

**文件**：`StoreMapper.java`

```java
/**
 * 通过 slug 查询店铺
 */
StoreEntity selectBySlug(@Param("slug") String slug);
```

**文件**：`StoreMapper.xml`

```xml
<select id="selectBySlug" resultType="com.casalyin.admin.module.busniess.store.domain.entity.StoreEntity">
    SELECT * FROM t_store WHERE slug = #{slug}
</select>
```

### 2.2 Service 层

**文件**：`StoreService.java`

```java
/**
 * 通过 slug 查询店铺详情
 */
public StoreVO getBySlug(String slug) {
    StoreEntity entity = storeMapper.selectBySlug(slug);
    if (entity == null) {
        return null;
    }
    // 转换为 VO，填充 VIP 信息等
    return convertToVO(entity);
}
```

### 2.3 Controller 层

**文件**：`StoreController.java`

```java
@GetMapping("/detail/by-slug/{slug}")
public ResponseDTO<StoreVO> getBySlug(@PathVariable String slug) {
    StoreVO store = storeService.getBySlug(slug);
    if (store == null) {
        return ResponseDTO.userErrorParam("店铺不存在");
    }
    return ResponseDTO.ok(store);
}
```

---

## 任务 3：slug 生成逻辑

### 3.1 工具类

**文件**：`SlugUtils.java`

```java
public class SlugUtils {
    /**
     * 生成 slug
     * 规则：小写、空格转连字符、移除特殊字符
     */
    public static String generateSlug(String name) {
        if (name == null || name.isEmpty()) {
            return null;
        }
        
        // 转小写
        String slug = name.toLowerCase();
        
        // 移除重音符号
        slug = Normalizer.normalize(slug, Normalizer.Form.NFD);
        slug = slug.replaceAll("\\p{M}", "");
        
        // 空格转连字符
        slug = slug.replaceAll("\\s+", "-");
        
        // 只保留字母、数字、连字符
        slug = slug.replaceAll("[^a-z0-9-]", "");
        
        // 移除多余的连字符
        slug = slug.replaceAll("-+", "-");
        slug = slug.replaceAll("^-|-$", "");
        
        return slug;
    }
    
    /**
     * 确保 slug 唯一性
     */
    public static String ensureUniqueSlug(String baseSlug, Function<String, Boolean> existsChecker) {
        String slug = baseSlug;
        int counter = 2;
        
        while (existsChecker.apply(slug)) {
            slug = baseSlug + "-" + counter;
            counter++;
        }
        
        return slug;
    }
}
```

### 3.2 在创建/更新时生成 slug

**文件**：`StoreService.java`

```java
// 创建店铺时
public void create(StoreAddForm form) {
    StoreEntity entity = new StoreEntity();
    // ... 其他字段
    
    // 生成 slug
    String baseSlug = SlugUtils.generateSlug(form.getStoreName());
    String slug = SlugUtils.ensureUniqueSlug(baseSlug, 
        s -> storeMapper.selectBySlug(s) != null);
    entity.setSlug(slug);
    
    storeMapper.insert(entity);
}

// 更新店铺名称时
public void update(Long storeId, StoreUpdateForm form) {
    StoreEntity entity = storeMapper.selectById(storeId);
    
    // 如果店铺名称改变，重新生成 slug
    if (!entity.getStoreName().equals(form.getStoreName())) {
        String baseSlug = SlugUtils.generateSlug(form.getStoreName());
        String slug = SlugUtils.ensureUniqueSlug(baseSlug, 
            s -> {
                StoreEntity existing = storeMapper.selectBySlug(s);
                return existing != null && !existing.getStoreId().equals(storeId);
            });
        entity.setSlug(slug);
    }
    
    // ... 其他更新逻辑
    storeMapper.updateById(entity);
}
```

---

## 任务 4：确保 VO 返回 slug

**文件**：`StoreVO.java`

```java
/**
 * 店铺 slug
 */
private String slug;
```

确保所有返回店铺详情的接口都包含 slug 字段。

---

## 验收标准

- [ ] 数据库添加了 slug 字段
- [ ] 现有店铺都有 slug
- [ ] 实现了 `/detail/by-slug/{slug}` API
- [ ] API 返回的数据包含 slug 字段
- [ ] 创建店铺时自动生成 slug
- [ ] 更新店铺名称时更新 slug
- [ ] slug 唯一性得到保证
- [ ] 测试通过

---

## 测试用例

```bash
# 1. 通过 slug 查询店铺
curl http://localhost:8690/api/store/detail/by-slug/tienda-agricola-san-jose

# 2. 创建店铺，检查是否自动生成 slug
# 3. 更新店铺名称，检查 slug 是否更新
# 4. 创建同名店铺，检查 slug 是否添加数字后缀
```

---

**预计时间**：2-3 小时
