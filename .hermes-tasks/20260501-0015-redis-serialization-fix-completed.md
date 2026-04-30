# Redis 序列化异常修复 - 已完成

## 问题描述

访问 `/api/store/public/detail/20` 时报 Redis 序列化异常：

```
org.springframework.data.redis.serializer.SerializationException: 
Cannot serialize; nested exception is 
org.springframework.core.serializer.support.SerializationFailedException: 
Failed to serialize object using DefaultSerializer
```

根本原因：`StoreVO` 类没有实现 `Serializable` 接口，Spring Cache 使用 JDK 序列化器无法序列化。

## 解决方案

采用**长期方案**：配置 Spring Cache 使用 JSON 序列化器而非 JDK 序列化器。

### 关键发现

1. **sa-base 已有完整配置**：
   - `RedisConfig.java` 配置了 `RedisTemplate` 使用 `Jackson2JsonRedisSerializer`
   - `CacheConfig.java` 配置了 `CacheManager` 使用 JSON 序列化器
   - `sa-base.yaml` 配置了 `spring.cache.type: redis`

2. **问题根源**：
   - `CacheConfig` 的 `@ConditionalOnProperty(name = "spring.cache.type", havingValue = "redis")` 要求配置存在
   - 但 Spring Boot 在加载 sa-base.yaml **之前**就评估了条件，导致 Bean 未创建
   - 结果：Spring Cache 使用了默认的 JDK 序列化器

### 修复步骤

在 `casalyin-admin/src/main/resources/dev/application.yaml` 中添加：

```yaml
spring:
  cache:
    type: redis
```

这确保了 `@ConditionalOnProperty` 在 Bean 创建时能正确评估。

## 验证结果

✅ 后端启动成功（PID 47414，端口 8690）
✅ 请求 `/api/store/public/detail/20` 成功返回数据
✅ 请求 `/api/store/public/detail/21` 成功返回数据
✅ 请求 `/api/store/public/detail/22` 成功返回数据
✅ 错误日志无新增序列化异常（最后错误时间 23:57:08）

## 影响范围

- 修改文件：`casalyin-admin/src/main/resources/dev/application.yaml`
- 影响：所有使用 `@Cacheable` 的方法现在使用 JSON 序列化器
- 优势：
  - 无需修改 VO 类（不需要实现 `Serializable`）
  - JSON 格式可读性强，便于调试
  - 跨语言兼容性好
  - 避免 JDK 序列化的版本兼容问题

## 后续建议

1. 同步修改生产环境配置（`prod/application.yaml`）
2. 考虑清理 Redis 中可能存在的旧格式缓存数据
3. 验证其他环境（test、staging）的配置一致性

---

完成时间：2026-05-01 00:15
修复人：Hermes
