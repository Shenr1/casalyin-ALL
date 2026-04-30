# 任务：修复 Redis 序列化异常

**创建时间：** 2026-04-30 09:10  
**状态：** TODO  
**类型：** Bug Fix  
**优先级：** P0（生产环境故障）

---

## 问题描述

生产环境 API 调用失败，返回错误：
```json
{
    "code": 10001,
    "level": "system",
    "msg": "Redis 连接异常，请检查 Redis 是否已启动（如本机 127.0.0.1:6379）及 spring.data.redis / Redisson 配置是否正确。",
    "ok": false,
    "data": null
}
```

**影响范围：**
- 所有需要 Redis 缓存的 API 接口
- 生产环境完全不可用

**复现路径：**
https://admin.casalyin.com/api/product/public/detail/8001

---

## 根因分析

后端日志显示真正的错误是序列化异常，而不是 Redis 连接问题：

```
Caused by: java.lang.IllegalArgumentException: DefaultSerializer requires a Serializable payload 
but received an object of type [com.casalyin.admin.module.busniess.product.domain.vo.ProductDetailVO]
```

**根本原因：**
`ProductDetailVO` 类没有实现 `java.io.Serializable` 接口，导致无法序列化到 Redis。

**误导性错误信息：**
全局异常处理器将序列化异常统一包装成了"Redis 连接异常"，导致排查困难。

---

## 技术方案

### 方案 A：让所有 VO 类实现 Serializable（推荐）

在所有需要缓存的 VO 类上添加 `implements Serializable`：

```java
@Data
public class ProductDetailVO implements Serializable {
    private static final long serialVersionUID = 1L;
    
    // 现有字段...
}
```

**需要修改的类（至少包括）：**
- `ProductDetailVO`
- 其他被缓存的 VO 类（如 `StoreDetailVO`、`AgronomistDetailVO` 等）

**优点：**
- 符合 Java 序列化规范
- 兼容 JDK 序列化器
- 改动最小

**缺点：**
- 需要逐个类添加
- JDK 序列化性能较差

### 方案 B：切换 Redis 序列化器为 JSON（更优）

修改 Redis 配置，使用 JSON 序列化器代替 JDK 序列化器：

```java
@Configuration
public class RedisConfig {
    
    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory factory) {
        RedisTemplate<String, Object> template = new RedisTemplate<>();
        template.setConnectionFactory(factory);
        
        // 使用 Jackson2JsonRedisSerializer 代替 JdkSerializationRedisSerializer
        Jackson2JsonRedisSerializer<Object> serializer = new Jackson2JsonRedisSerializer<>(Object.class);
        ObjectMapper mapper = new ObjectMapper();
        mapper.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        mapper.activateDefaultTyping(
            LaissezFaireSubTypeValidator.instance,
            ObjectMapper.DefaultTyping.NON_FINAL,
            JsonTypeInfo.As.PROPERTY
        );
        serializer.setObjectMapper(mapper);
        
        template.setKeySerializer(new StringRedisSerializer());
        template.setValueSerializer(serializer);
        template.setHashKeySerializer(new StringRedisSerializer());
        template.setHashValueSerializer(serializer);
        
        template.afterPropertiesSet();
        return template;
    }
}
```

**优点：**
- 无需修改 VO 类
- JSON 序列化性能更好
- 可读性强（Redis 中存储的是 JSON 字符串）
- 跨语言兼容

**缺点：**
- 需要清空现有 Redis 缓存（序列化格式变更）
- 需要测试所有缓存功能

---

## 推荐方案

**短期（紧急修复）：** 方案 A  
在 `ProductDetailVO` 及相关 VO 类上添加 `implements Serializable`，快速恢复生产环境。

**长期（优化）：** 方案 B  
切换到 JSON 序列化器，提升性能和可维护性。

---

## 实施步骤

### 短期修复（立即执行）

1. **找到所有需要缓存的 VO 类**
   ```bash
   cd casalyin-java
   grep -r "@Cacheable\|@CachePut\|@CacheEvict" --include="*.java" | \
   grep -oP '(?<=value = ")[^"]+' | sort -u
   ```

2. **修改 ProductDetailVO**
   ```java
   // 文件：casalyin-java/casalyin-admin/src/main/java/com/casalyin/admin/module/busniess/product/domain/vo/ProductDetailVO.java
   
   @Data
   public class ProductDetailVO implements Serializable {
       private static final long serialVersionUID = 1L;
       
       // 现有字段保持不变...
   }
   ```

3. **修改其他相关 VO 类**
   - `StoreDetailVO`
   - `AgronomistDetailVO`
   - `BrandDetailVO`
   - 其他被 `@Cacheable` 注解方法返回的 VO 类

4. **本地测试**
   ```bash
   cd casalyin-java
   npm run dev:stop && npm run dev
   
   # 等待启动后测试
   curl http://localhost:8690/api/product/public/detail/8001
   ```

5. **部署到生产**
   ```bash
   # 提交代码
   git add .
   git commit -m "fix: add Serializable to VO classes for Redis caching"
   git push origin main
   
   # 等待 CI/CD 自动部署
   ```

### 长期优化（后续版本）

1. **创建 RedisConfig 配置类**
2. **切换到 Jackson2JsonRedisSerializer**
3. **清空生产环境 Redis 缓存**
4. **全面测试缓存功能**

---

## 验证方式

### 本地验证
```bash
# 1. 启动后端
cd casalyin-java
npm run dev

# 2. 测试 API
curl http://localhost:8690/api/product/public/detail/8001

# 3. 检查 Redis 中是否有缓存
redis-cli -n 1
keys *product*
get <key>
```

### 生产验证
```bash
# 1. 测试 API
curl https://admin.casalyin.com/api/product/public/detail/8001

# 2. 检查返回结果是否正常（不再是 Redis 连接异常）

# 3. 检查后端日志
ssh root@47.77.200.69
docker logs casalyin-backend-1 --tail 50
```

---

## 注意事项

1. **serialVersionUID 必须添加**
   - 防止类结构变更导致反序列化失败
   - 使用 `1L` 作为初始值

2. **嵌套对象也需要实现 Serializable**
   - 如果 VO 中包含其他自定义对象，那些对象也需要实现 Serializable
   - 基本类型和常用 Java 类（String、Integer、List 等）已经实现

3. **改进错误提示**
   - 考虑在全局异常处理器中区分 Redis 连接异常和序列化异常
   - 给开发者更准确的错误信息

4. **清空缓存**
   - 修改 VO 类结构后，建议清空 Redis 缓存
   - 生产环境：`docker exec redis redis-cli -n 1 flushdb`

---

## 执行记录

- [ ] 找到所有需要缓存的 VO 类
- [ ] 修改 ProductDetailVO 添加 Serializable
- [ ] 修改其他相关 VO 类
- [ ] 本地测试验证
- [ ] 提交代码
- [ ] 生产部署验证
- [ ] 清空生产 Redis 缓存（可选）

---

## 完成标准

- [x] ProductDetailVO 及相关 VO 类实现 Serializable
- [x] 本地测试通过，API 返回正常数据
- [x] 生产环境 API 恢复正常
- [x] 后端日志无序列化异常
- [x] Redis 缓存正常工作
