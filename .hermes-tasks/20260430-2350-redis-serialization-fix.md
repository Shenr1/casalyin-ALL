# Redis 序列化异常修复

## 问题描述

访问 `/api/store/public/detail/20` 时报错：

```
org.springframework.data.redis.serializer.SerializationException: Cannot serialize
Caused by: java.lang.IllegalArgumentException: DefaultSerializer requires a Serializable payload 
but received an object of type [com.casalyin.admin.module.busniess.store.domain.vo.StoreVO]
```

## 根本原因

`StoreVO` 类没有实现 `java.io.Serializable` 接口，导致 Spring Data Redis 的 JDK 序列化器无法序列化对象。

## 解决方案（二选一）

### 方案 1：快速修复（推荐用于紧急修复）

给 `StoreVO` 添加 `Serializable` 接口：

```java
// 文件：casalyin-admin/src/main/java/com/casalyin/admin/module/busniess/store/domain/vo/StoreVO.java

@Data
public class StoreVO implements java.io.Serializable {
    private static final long serialVersionUID = 1L;
    // ... 其他代码不变
}
```

### 方案 2：配置 JSON 序列化器（推荐用于长期方案）

修改 Redis 配置，使用 JSON 序列化器替代 JDK 序列化器：

```java
// 新建文件：casalyin-admin/src/main/java/com/casalyin/admin/config/RedisConfig.java

package com.casalyin.admin.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.serializer.GenericJackson2JsonRedisSerializer;
import org.springframework.data.redis.serializer.StringRedisSerializer;

@Configuration
public class RedisConfig {

    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory factory) {
        RedisTemplate<String, Object> template = new RedisTemplate<>();
        template.setConnectionFactory(factory);
        
        // 使用 JSON 序列化器
        GenericJackson2JsonRedisSerializer jsonSerializer = new GenericJackson2JsonRedisSerializer();
        StringRedisSerializer stringSerializer = new StringRedisSerializer();
        
        template.setKeySerializer(stringSerializer);
        template.setValueSerializer(jsonSerializer);
        template.setHashKeySerializer(stringSerializer);
        template.setHashValueSerializer(jsonSerializer);
        
        template.afterPropertiesSet();
        return template;
    }
}
```

**方案 2 的优势：**
- 避免 JDK 序列化的版本兼容性问题
- JSON 格式可读性强，便于调试
- 不需要每个 VO 类都实现 Serializable

## 验证步骤

1. 重启后端服务
2. 访问 `http://localhost:5173/api/store/public/detail/20`
3. 确认不再报 SerializationException

## 影响范围

- 本地开发环境（dev profile）
- 可能影响所有使用 Redis 缓存的 VO 类

## 优先级

中等 - 影响本地开发体验，但不影响生产环境（生产环境可能已有不同配置）

---

**创建时间：** 2026-04-30 23:50  
**创建者：** Hermes  
**状态：** 待处理
