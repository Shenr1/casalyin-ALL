# Casalyin 本地开发环境诊断与修复

> **重要**: 启动 Spring Boot 后端时，请先阅读 [Spring Boot 本地启动指南](./spring-boot-startup-guide.md)，避免陷入无限循环重试。

## 适用场景

当遇到以下情况时，参考本文档：
- 本地服务启动失败
- 接口返回 500 错误
- 本地环境配置问题
- 服务启动后无法访问

## 项目信息

- **项目路径**: ~/Documents/Project/casalyin-ALL/
- **技术栈**: 
  - 后端: Spring Boot + Java 17 + Maven
  - 前台: Next.js + React + TypeScript
  - 后台: Vite + React + TypeScript
  - 缓存: Redis (Docker)
  - 数据库: MySQL (阿里云 RDS)

## 服务架构

```
Spring Boot 后端  → 端口 8690  → ~/casalyin-java/casalyin-admin
Next.js 前台     → 端口 3000  → ~/casalyin-Headless
Vite 后台管理    → 端口 5173  → ~/casalyin-server
Redis 缓存       → 端口 6379  → Docker 容器
MySQL 数据库     → 远程 RDS   → rm-rj9575p29a07omi0q.mysql.rds.aliyuncs.com:3306
```

## 诊断流程

按以下顺序检查，发现问题立即修复：

### 1. 检查 Java 环境

```bash
# 检查 Java 版本
java -version 2>&1 | grep "openjdk 17"

# 检查 JAVA_HOME
echo $JAVA_HOME | grep "openjdk@17"
```

**预期结果**: 
- Java 版本为 17.x
- JAVA_HOME 指向 `/opt/homebrew/Cellar/openjdk@17/17.0.19/libexec/openjdk.jdk/Contents/Home`

**异常处理**:
```bash
# 如果 Java 未安装或版本不对
brew install openjdk@17

# 设置 JAVA_HOME（临时）
export JAVA_HOME=/opt/homebrew/Cellar/openjdk@17/17.0.19/libexec/openjdk.jdk/Contents/Home
```

### 2. 检查 Node.js 环境

```bash
# 检查 Node.js 版本
node -v

# 检查 npm 版本
npm -v
```

**预期结果**: Node.js >= 18.x

**异常处理**:
```bash
# 如果 Node.js 未安装
brew install node
```

### 3. 检查 Redis 容器

```bash
# 检查 Redis 是否运行
docker ps | grep redis
```

**预期结果**: Redis 容器状态为 `Up`

**异常处理**:
```bash
# 如果 Redis 未运行，尝试启动
docker start redis

# 如果容器不存在，创建新容器
if ! docker ps -a | grep -q redis; then
    docker run -d --name redis -p 6379:6379 redis:alpine
fi

# 验证 Redis 可访问
redis-cli ping
```

### 4. 检查端口占用情况

```bash
# 检查关键端口是否被占用
lsof -i :8690  # 后端
lsof -i :3000  # Next.js
lsof -i :5173  # Vite
lsof -i :6379  # Redis
```

**异常处理**:
- 如果端口被占用且不是目标服务，kill 进程
- 如果是目标服务，说明服务已启动，跳过启动步骤

### 5. 检查环境变量配置

#### 5.1 检查 Next.js 环境变量

```bash
# 检查 .env.local 是否存在
ls -la ~/Documents/Project/casalyin-ALL/casalyin-Headless/.env.local
```

**预期结果**: 文件存在且包含以下内容：
```
API_INTERNAL_URL=http://localhost:8690/api
NEXT_PUBLIC_API_URL=http://localhost:8690/api
```

**异常处理**:
```bash
# 如果文件不存在，自动创建
cat > ~/Documents/Project/casalyin-ALL/casalyin-Headless/.env.local << 'EOF'
# 本地开发环境配置
# 服务端（SSR）使用的后端 API 地址
API_INTERNAL_URL=http://localhost:8690/api

# 客户端（CSR）使用的后端 API 地址
NEXT_PUBLIC_API_URL=http://localhost:8690/api
EOF

echo "✅ 已创建 .env.local 文件"
```

**关键点**:
- 后端有 `server.servlet.context-path: /api` 配置
- 环境变量应该配置为 `http://localhost:8690/api`
- 修改后**必须重启 Next.js**

#### 5.2 检查后台管理环境变量（Google Maps API Key）

```bash
# 检查 .env 文件中是否配置了 Google Maps API Key
grep "VITE_GOOGLE_MAPS_API_KEY" ~/Documents/Project/casalyin-ALL/casalyin-server/.env
```

**预期结果**: 输出类似 `VITE_GOOGLE_MAPS_API_KEY=AIzaSy...`

**异常处理**:
```bash
# 如果未配置，添加 Google Maps API Key
cat >> ~/Documents/Project/casalyin-ALL/casalyin-server/.env << 'EOF'

# Google Maps API Key (用于地址和城市选择组件)
VITE_GOOGLE_MAPS_API_KEY=你的API密钥
EOF

echo "✅ 已添加 Google Maps API Key 配置"
echo "⚠️  需要重启 Vite 服务才能生效"
```

**关键点**:
- 后台管理使用两个地址选择组件：`PlacesCityAutocomplete`（城市选择）和 `PlacesAddressAutocomplete`（地址选择）
- API Key 需要启用：Places API、Geocoding API、Maps JavaScript API
- 配置后必须重启 Vite 服务

### 6. 检查数据库连接

```bash
# 测试数据库连接（使用 MySQL 客户端）
mysql -h rm-rj9575p29a07omi0q.mysql.rds.aliyuncs.com -P 3306 -u casalyin -p -e "SELECT 1" 2>&1
```

**预期结果**: 连接成功

**异常处理**:
- 如果连接失败，检查网络和数据库密码
- 确认 `casalyin-java/casalyin-admin/src/main/resources/application-dev.yaml` 中的数据库配置正确

### 7. 启动后端服务

```bash
cd ~/Documents/Project/casalyin-ALL/casalyin-java/casalyin-admin

# 设置 JAVA_HOME
export JAVA_HOME=/opt/homebrew/Cellar/openjdk@17/17.0.19/libexec/openjdk.jdk/Contents/Home

# 启动 Spring Boot（后台运行）
../mvnw spring-boot:run -Dspring-boot.run.profiles=dev -Dspring.devtools.restart.enabled=false > /tmp/casalyin-backend.log 2>&1 &

# 记录 PID
BACKEND_PID=$!
echo "后端 PID: $BACKEND_PID"
```

**启动验证**:
```bash
# ⚠️ 重要：后端启动需要 2-3 分钟完成数据库初始化
# 不要过早测试，否则会误判为启动失败

# 1. 等待 Tomcat 启动信号（约 30-40 秒）
echo "等待 Tomcat 启动..."
timeout 60 tail -f /tmp/casalyin-backend.log | grep -m 1 "Tomcat started on port"

# 2. 继续等待数据库初始化完成（约 1-2 分钟）
echo "Tomcat 已启动，等待数据库初始化完成..."
sleep 90

# 3. 检查端口是否真正监听（这是最可靠的验证方式）
lsof -i :8690

# 4. 测试 API 可访问性
curl -s -X POST -H "Content-Type: application/json" -d '{"pageNum":1,"pageSize":1}' \
  http://localhost:8690/api/product/public/filter | head -c 100
```

**预期结果**: 
- 日志显示 "Tomcat started on port 8690"
- 端口 8690 被 java 进程监听
- API 返回 `{"code":0,"msg":"操作成功"...}`

**关键陷阱**:
- ❌ **不要**看到 "Tomcat started" 就立即测试 API
- ❌ **不要**在端口未监听时反复重启服务
- ✅ **必须**等待数据库初始化完成（Flyway 迁移 + 权限数据插入）
- ✅ **必须**用 `lsof -i :8690` 确认端口真正监听后再测试
- ⏱️ **总启动时间**: 2-3 分钟

### 8. 启动 Next.js 前台

```bash
cd ~/Documents/Project/casalyin-ALL/casalyin-Headless

# 检查依赖是否安装
if [ ! -d "node_modules" ]; then
    echo "📦 安装 Next.js 依赖..."
    npm install
fi

# 启动 Next.js（后台运行）
npm run dev > /tmp/casalyin-nextjs.log 2>&1 &

# 记录 PID
NEXTJS_PID=$!
echo "Next.js PID: $NEXTJS_PID"
```

**启动验证**:
```bash
# 等待 10 秒后检查
sleep 10

# 测试首页可访问性
curl -s -o /dev/null -w '%{http_code}' http://localhost:3000

# 测试 API 代理
curl -X POST -H "Content-Type: application/json" -d '{"pageNum":1,"pageSize":10}' \
  http://localhost:3000/api/product/filter 2>&1 | grep -o '"code":[0-9]*'
```

**预期结果**:
- 首页返回 200
- API 代理返回 `"code":0`（成功）

### 9. 启动 Vite 后台管理

```bash
cd ~/Documents/Project/casalyin-ALL/casalyin-server

# 检查依赖是否安装
if [ ! -d "node_modules" ]; then
    echo "📦 安装 Vite 依赖..."
    npm install
fi

# 检查是否配置了 Google Maps API Key
if ! grep -q "VITE_GOOGLE_MAPS_API_KEY" .env; then
    echo "⚠️  警告：未配置 Google Maps API Key，地址和城市选择功能将无法使用"
    echo "请在 .env 文件中添加：VITE_GOOGLE_MAPS_API_KEY=你的API密钥"
fi

# 启动 Vite（后台运行）
npm run dev > /tmp/casalyin-vite.log 2>&1 &

# 记录 PID
VITE_PID=$!
echo "Vite PID: $VITE_PID"
```

**启动验证**:
```bash
# 等待 5 秒后检查
sleep 5

# 测试可访问性
curl -s -o /dev/null -w '%{http_code}' http://localhost:5173
```

**预期结果**: 返回 200

### 10. 完整健康检查

所有服务启动后，执行完整验证：

```bash
echo "=== 服务健康检查 ==="

# 后端 API
echo -n "后端 API (8690): "
curl -s -o /dev/null -w '%{http_code}' http://localhost:8690/api/product/list
echo ""

# Next.js 首页
echo -n "Next.js 首页 (3000): "
curl -s -o /dev/null -w '%{http_code}' http://localhost:3000
echo ""

# Next.js API 代理
echo -n "Next.js API 代理: "
curl -X POST -H "Content-Type: application/json" -d '{"pageNum":1,"pageSize":10}' \
  http://localhost:3000/api/product/filter 2>&1 | grep -o '"code":[0-9]*'
echo ""

# Vite 后台
echo -n "Vite 后台 (5173): "
curl -s -o /dev/null -w '%{http_code}' http://localhost:5173
echo ""

# Redis
echo -n "Redis (6379): "
redis-cli ping
echo ""
```

## 常见问题和解决方案

### 问题 1: Next.js API 返回 500 "fetch failed"

**现象**: 
```bash
curl -X POST http://localhost:3000/api/product/filter
# 返回: {"code":500,"msg":"fetch failed"}
```

**根因**: `.env.local` 文件缺失或配置错误

**解决**:
```bash
# 创建配置文件
cat > ~/Documents/Project/casalyin-ALL/casalyin-Headless/.env.local << 'EOF'
# 本地开发环境配置
API_INTERNAL_URL=http://localhost:8690/api
NEXT_PUBLIC_API_URL=http://localhost:8690/api
EOF

# 重启 Next.js（必须！）
pkill -f "next-server"
cd ~/Documents/Project/casalyin-ALL/casalyin-Headless
npm run dev > /tmp/casalyin-nextjs.log 2>&1 &
```

### 问题 2: 后端"假启动" - Tomcat 启动但端口未监听

**现象**: 
- 日志显示 "Tomcat started on port 8690"
- 但 `lsof -i :8690` 返回空
- `curl http://localhost:8690/api/...` 连接失败

**根因**: Spring Boot 启动分两阶段：
1. Tomcat 启动 (35秒)
2. 数据库初始化 (90-120秒)

**解决**:
```bash
# ✅ 正确做法：耐心等待初始化完成
echo "等待后端初始化完成（约 2-3 分钟）..."
sleep 150

# 验证端口监听
lsof -i :8690

# 测试 API
curl -s -X POST -H "Content-Type: application/json" -d '{"pageNum":1,"pageSize":1}' \
  http://localhost:8690/api/product/public/filter | python3 -m json.tool | head -10
```

### 问题 3: Google Maps API 调用失败

**现象**: 
- 后台管理中的地址选择或城市选择组件无法使用
- 浏览器控制台显示 `REQUEST_DENIED` 错误

**根因**: `.env` 文件中缺少 `VITE_GOOGLE_MAPS_API_KEY` 配置

**解决**:
```bash
# 添加 Google Maps API Key
cat >> ~/Documents/Project/casalyin-ALL/casalyin-server/.env << 'EOF'

# Google Maps API Key
VITE_GOOGLE_MAPS_API_KEY=你的API密钥
EOF

# 重启 Vite 服务（必须！）
pkill -f "vite"
cd ~/Documents/Project/casalyin-ALL/casalyin-server
npm run dev > /tmp/casalyin-vite.log 2>&1 &
```

## 快速命令参考

### 启动所有服务

```bash
# 1. 启动 Redis
docker start redis

# 2. 启动后端
cd ~/Documents/Project/casalyin-ALL/casalyin-java/casalyin-admin
export JAVA_HOME=/opt/homebrew/Cellar/openjdk@17/17.0.19/libexec/openjdk.jdk/Contents/Home
../mvnw spring-boot:run -Dspring-boot.run.profiles=dev &

# 3. 启动 Next.js
cd ~/Documents/Project/casalyin-ALL/casalyin-Headless
npm run dev &

# 4. 启动 Vite
cd ~/Documents/Project/casalyin-ALL/casalyin-server
npm run dev &
```

### 停止所有服务

```bash
# 查找所有相关进程
ps aux | grep -E "(spring-boot|next-server|vite)" | grep -v grep

# Kill 所有进程
kill <PID1> <PID2> <PID3>

# 停止 Redis
docker stop redis
```

### 查看日志

```bash
# 后端日志
tail -f /tmp/casalyin-backend.log

# Next.js 日志
tail -f /tmp/casalyin-nextjs.log

# Vite 日志
tail -f /tmp/casalyin-vite.log
```

## 注意事项

1. **环境变量修改后必须重启服务**
2. **后端启动需要 2-3 分钟**，不要过早测试
3. **后端启动的正确验证方式**: 用 `lsof -i :8690` 确认端口监听
4. **Next.js 修改 .env.local 后必须完全重启**
5. **Redis 必须先启动**，否则后端会启动失败
6. **遇到问题时先诊断、制定计划、再执行**，避免盲目重试

## 访问地址

- 后端 API 文档: http://localhost:8690/api/doc.html
- 前台: http://localhost:3000
- 后台管理: http://localhost:5173
