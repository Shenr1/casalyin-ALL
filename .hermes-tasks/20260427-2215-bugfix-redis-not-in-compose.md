# [bugfix] Redis 未纳入 Docker Compose 管理导致服务重启后无法自动恢复

**创建时间：** 2026-04-27 22:15  
**创建者：** Hermes  
**优先级：** 🔴 高  
**状态：** DONE

---

## 问题描述

生产服务器重启后，后端服务无法启动，报错：
```
Unable to connect to Redis server: host.docker.internal/172.17.0.1:6379
```

经诊断发现：
1. Redis 容器已停止（4 小时前收到 SIGTERM 信号）
2. Redis 容器不在 docker-compose.yml 中，是独立启动的
3. 服务器重启后 Redis 不会自动启动
4. Redis 在 `bridge` 网络，后端在 `casalyin_default` 网络，网络隔离

## 根本原因

1. **Redis 未纳入 Docker Compose 管理**
   - 当前 Redis 是手动启动的独立容器
   - 不在 `casalyin-Headless/deploy/docker-compose.yml` 中
   - 服务器重启后不会自动启动

2. **Docker 网络配置问题**
   - Redis 在默认 `bridge` 网络
   - 后端在 `casalyin_default` 网络
   - 两者无法直接通信

3. **后端配置使用 host.docker.internal**
   - 环境变量：`REDIS_HOST=host.docker.internal`
   - 这在 Linux 上不可靠（仅适用于 Docker Desktop for Mac/Windows）

## 影响范围

- **影响服务：** casalyin-backend（后端 API）
- **严重程度：** 🔴 高（导致整个后端服务无法启动）
- **临时修复：** ✅ 已完成
  - 已执行：`docker start redis`
  - 已执行：`docker network connect casalyin_default redis`
  - 已执行：`docker restart casalyin-backend-1`
  - 当前所有服务正常运行

## 需要修改的文件

- [ ] `casalyin-Headless/deploy/docker-compose.yml`
- [ ] `casalyin-java/.env.prod`（可选，如果需要修改 Redis 配置）

## 修改方案

### 方案 1：将 Redis 加入 Docker Compose（推荐）

**修改文件：** `casalyin-Headless/deploy/docker-compose.yml`

在 `services:` 下添加 Redis 服务：

```yaml
services:
  # 无头电商前台（Next.js）
  headless:
    # ... 现有配置 ...

  # 管理后台前端（Vite SPA，Nginx 托管）
  server:
    # ... 现有配置 ...

  # 后端 API（Spring Boot）
  backend:
    build:
      context: ../../casalyin-java
      dockerfile: Dockerfile
    image: casalyin-backend:latest
    container_name: casalyin-backend
    restart: unless-stopped
    ports:
      - "1025:1025"
    environment:
      - REDIS_HOST=redis          # 👈 使用服务名称
      - REDIS_PORT=6379
    depends_on:
      - redis                      # 👈 确保 Redis 先启动

  # Redis 缓存服务
  redis:
    image: redis:7-alpine
    container_name: redis
    restart: unless-stopped
    ports:
      - "6379:6379"
    volumes:
      - redis-data:/data
    command: redis-server --appendonly yes
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 3s
      retries: 3

# 在文件末尾添加 volumes 定义
volumes:
  redis-data:
    driver: local
```

**关键改动：**
1. 添加 `redis` 服务定义
2. 在 `backend` 中添加 `REDIS_HOST=redis` 环境变量
3. 在 `backend` 中添加 `depends_on: redis`
4. 添加 `redis-data` volume 持久化数据
5. 添加健康检查

### 方案 2：仅修改后端配置（不推荐）

如果暂时不想改 docker-compose.yml，可以只修改后端配置：

**修改文件：** `casalyin-java/.env.prod`

```bash
# 改为使用 Redis 容器名称
REDIS_HOST=redis
REDIS_PORT=6379
```

**缺点：**
- Redis 仍然需要手动启动
- 服务器重启后仍需手动干预
- 不是长期解决方案

## 验证方式

### 本地验证（修改后）

```bash
# 1. 验证 docker-compose 配置语法
cd casalyin-Headless/deploy
docker compose config

# 2. 停止现有容器
docker compose down

# 3. 删除旧的独立 Redis 容器（可选）
docker rm -f redis

# 4. 启动所有服务
docker compose up -d

# 5. 检查所有容器状态
docker compose ps

# 6. 查看后端日志
docker compose logs backend

# 7. 测试后端 API
curl http://localhost:1025/api/
```

### 生产环境验证（部署后）

Hermes 会在服务器上执行以下验证：

```bash
# 1. 检查所有容器状态
docker ps

# 2. 测试 Redis 连接
docker exec redis redis-cli ping

# 3. 测试后端 API
curl http://localhost:1025/api/

# 4. 查看后端日志（确认无 Redis 连接错误）
docker logs casalyin-backend-1 --tail 50

# 5. 模拟服务器重启（重启所有容器）
docker compose restart

# 6. 确认所有服务自动恢复
docker compose ps
```

## 相关资源

### 服务器信息
- 公网 IP：47.77.200.69
- 操作系统：Alibaba Cloud Linux 3.2104
- Docker Compose 位置：`/opt/casalyin/casalyin-Headless/deploy/`

### 错误日志片段

```
Caused by: org.redisson.client.RedisConnectionException: 
Unable to connect to Redis server: host.docker.internal/172.17.0.1:6379

Caused by: java.net.ConnectException: Connection refused
```

### 当前容器状态（临时修复后）

```
CONTAINER ID   IMAGE                      STATUS          PORTS
63f7e946c522   casalyin-headless:latest   Up 8 minutes    0.0.0.0:3000->3000/tcp
294cbc0cd237   casalyin-server:latest     Up 8 minutes    0.0.0.0:8080->80/tcp
3e820f3a9548   casalyin-backend:latest    Up 20 seconds   0.0.0.0:1025->1025/tcp
2fb6f745dc85   redis:7-alpine             Up 28 seconds   0.0.0.0:6379->6379/tcp
```

---

## 执行记录

### Claude CLI 执行
- [x] 已读取任务
- [x] 已修改 docker-compose.yml
- [x] 已添加 Redis 服务配置
- [x] 已修改 backend 环境变量
- [x] 已本地验证配置
- [x] 已提交 Git
- [ ] 已触发部署

**执行时间：** 2026-04-28 00:12  
**执行者：** Claude CLI  
**Git Commit：** 0cb6306

**完成的修改：**
1. 在 `docker-compose.yml` 和 `docker-compose.prod.yml` 中添加了 Redis 服务
2. 修改 backend 环境变量：`REDIS_HOST=redis`（替代 `host.docker.internal`）
3. 添加了 `depends_on: redis` 确保启动顺序
4. 添加了 Redis 数据持久化 volume (`redis-data`)
5. 添加了健康检查确保 Redis 正常运行

### Hermes 验证
- [ ] 已验证部署
- [ ] Redis 容器正常运行
- [ ] 后端连接 Redis 成功
- [ ] 所有服务状态正常
- [ ] 模拟重启测试通过

**验证时间：** _待填写_  
**验证结果：** _待填写_

---

## 备注

### 临时修复措施（已执行）
```bash
docker start redis
docker network connect casalyin_default redis
docker restart casalyin-backend-1
```

### 为什么需要永久修复
1. 当前是临时方案，服务器重启后问题会再次出现
2. 需要将 Redis 纳入 Docker Compose 统一管理
3. 确保服务器重启后所有服务自动恢复
4. 避免每次都需要手动干预

### 注意事项
1. 修改后需要先在本地/测试环境验证
2. 部署时会短暂中断服务（约 30 秒）
3. Redis 数据会保留（使用 volume 持久化）
4. 建议在低峰期部署
