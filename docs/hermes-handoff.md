# Hermes ↔ Claude CLI 对接文档

> **用途**：当 Hermes 发现需要修改代码的问题时，通过此文档告知 Claude CLI 团队如何修复
> 
> **使用方式**：在 Claude CLI 中执行 `cat docs/hermes-handoff.md` 读取此文档

---

## 📋 当前待修复问题

### 问题 #1：Redis 未纳入 Docker Compose 管理

**现状：**
- Redis 容器是独立启动的（不在 docker-compose.yml 中）
- 容器名称：`redis`
- 镜像：`redis:7-alpine`
- 端口：6379
- 网络：需要同时在 `bridge` 和 `casalyin_default` 网络

**问题：**
- 服务器重启后 Redis 不会自动启动
- 导致后端无法连接 Redis，启动失败
- 需要手动执行 `docker start redis` 和网络连接

**需要修改的文件：**
`casalyin-Headless/deploy/docker-compose.yml`

**修改方案：**
在 `services:` 下添加 Redis 服务配置：

```yaml
services:
  # ... 现有的 headless, server, backend 服务 ...

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
    networks:
      - default

# 在文件末尾添加 volumes 定义
volumes:
  redis-data:
    driver: local
```

**验证方式：**
```bash
# 在服务器上执行
cd /opt/casalyin/casalyin-Headless/deploy
docker compose down
docker compose up -d
docker ps  # 确认所有 4 个容器都在运行
```

---

### 问题 #2：后端 Redis 配置使用 host.docker.internal

**现状：**
- 后端容器环境变量：`REDIS_HOST=host.docker.internal`
- 这在 Linux 上不可靠（`host.docker.internal` 是 Docker Desktop for Mac/Windows 的特性）

**问题：**
- 在 Linux 服务器上，`host.docker.internal` 可能无法解析
- 导致后端无法连接 Redis

**需要修改的文件：**
`casalyin-Headless/deploy/docker-compose.yml`

**修改方案：**
在 `backend` 服务中添加环境变量，使用 Docker 服务名称：

```yaml
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
      - REDIS_HOST=redis          # 使用服务名称而不是 host.docker.internal
      - REDIS_PORT=6379
    depends_on:
      - redis                      # 确保 Redis 先启动
```

**或者**（如果后端代码中使用 Spring Boot 配置）：

在 `casalyin-java/.env.prod` 或 `application-prod.yml` 中修改：

```yaml
spring:
  redis:
    host: redis  # 改为服务名称
    port: 6379
```

---

### 问题 #3：缺少数据库服务配置

**现状：**
- 未在 docker-compose.yml 中找到 PostgreSQL/MySQL 配置
- 后端日志显示有 `DataSourceConfig` 但未找到数据库容器

**待确认：**
1. 数据库是外部 RDS 还是本地容器？
2. 如果是本地容器，需要添加到 docker-compose.yml
3. 如果是 RDS，需要确认连接配置是否正确

**需要 Hermes 进一步诊断：**
- 查看后端完整日志中的数据库连接信息
- 检查是否有外部 RDS 配置
- 确认数据库类型（PostgreSQL/MySQL）

---

## 🔄 工作流程建议

### Hermes 的职责（运维）
1. 监控服务器状态
2. 诊断问题根因
3. 临时修复（重启服务、修复网络）
4. 更新此文档，告知 Claude CLI 需要改什么代码
5. 设置自动化监控和告警

### Claude CLI 的职责（开发）
1. 读取任务文档（.hermes-tasks/*.md 或 docs/hermes-handoff.md）
2. 修改代码和配置文件
3. 本地 commit（不要 push）
4. 更新任务状态为 DONE（如果是 .hermes-tasks/ 任务）
5. 通知用户任务已完成

### 协作流程
```
1. Hermes 发现问题
   ↓
2. Hermes 临时修复（让服务先跑起来）
   ↓
3. Hermes 创建任务文件（.hermes-tasks/*.md）
   ↓
4. 用户在 Claude CLI 中：读取任务文件
   ↓
5. Claude CLI 团队修改代码并 commit（不 push）
   ↓
6. 更新任务状态为 DONE
   ↓
7. Hermes 验证 → git push → 等待 CI/CD 部署 → 验证部署 → 归档任务
```

---

## 📊 当前服务器状态（由 Hermes 维护）

**最后更新：2026-04-27 22:14**

### 服务状态
- ✅ casalyin-backend（端口 1025）- 正常
- ✅ casalyin-server（端口 8080）- 正常  
- ✅ casalyin-headless（端口 3000）- 正常
- ✅ Redis（端口 6379）- 正常（临时修复）

### 临时修复措施
- 已执行：`docker start redis`
- 已执行：`docker network connect casalyin_default redis`
- 已执行：`docker restart casalyin-backend-1`

### 待解决问题
- [ ] 问题 #1：Redis 未纳入 Docker Compose 管理
- [ ] 问题 #2：后端 Redis 配置使用 host.docker.internal
- [ ] 问题 #3：缺少数据库服务配置（待确认）

---

## 🚀 快速命令参考

### 在 Claude CLI 中读取此文档
```bash
cat docs/hermes-handoff.md
```

### 修改后验证
```bash
# 本地测试 docker-compose 配置
cd casalyin-Headless/deploy
docker compose config  # 验证语法

# 提交代码（不要 push）
git add casalyin-Headless/deploy/docker-compose.yml
git commit -m "fix: 将 Redis 纳入 Docker Compose 管理，修复网络配置"

# 更新任务状态为 DONE（如果是 .hermes-tasks/ 任务）
# 然后通知用户任务已完成
```

### Hermes 验证和部署
用户通知 Hermes 后：
```
Hermes 会：
1. git pull 拉取 commit
2. 验证代码变更合理性
3. git push origin main
4. 等待 CI/CD 自动部署（约 3-5 分钟）
5. 验证部署结果（容器状态、API 接口、数据库连接）
6. 验证通过 → 归档任务
7. 验证失败 → 更新任务状态为 PENDING，通知用户继续修复
```

---

## 📝 问题追踪历史

### 2026-04-27
- **发现**：Redis 容器停止导致后端无法启动
- **根因**：Redis 不在 docker-compose.yml 中，重启后未自动启动
- **临时修复**：手动启动 Redis 并连接网络
- **待修复**：将 Redis 加入 docker-compose.yml

---

## 💡 最佳实践

1. **所有服务都应该在 docker-compose.yml 中定义**
   - 便于统一管理
   - 自动处理依赖关系
   - 服务器重启后自动恢复

2. **使用 Docker 服务名称而不是 IP 或 host.docker.internal**
   - 跨平台兼容
   - 自动 DNS 解析
   - 网络隔离更安全

3. **设置 restart: unless-stopped**
   - 服务器重启后自动启动
   - 异常退出后自动重启
   - 手动停止时不会自动启动

4. **使用 depends_on 声明依赖关系**
   - 确保启动顺序正确
   - 避免竞态条件

---

**文档维护者：Hermes**  
**最后更新：2026-04-27 22:15**
