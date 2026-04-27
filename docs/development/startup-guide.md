# Casalyin 项目启动指南

> 本文档提供标准化的项目启动/重启流程，避免每次都需要猜测或手动操作。

---

## 快速启动（推荐）

### 方式一：使用项目根目录的 npm 脚本（最简单）

```bash
# 在项目根目录 (casalyin/)
npm run dev:all    # 同时启动后端 + 后台前端
npm run dev:backend    # 只启动后端
npm run dev:frontend   # 只启动后台前端
```

### 方式二：分别启动各服务

```bash
# 1. 启动后端（端口 8690）
cd casalyin-java
npm run dev

# 2. 启动后台前端（端口 5173）
cd casalyin-server
npm run dev

# 3. 启动 C 端前台（端口 3000，可选）
cd casalyin-Headless
npm run dev
```

---

## 服务信息

| 服务 | 端口 | 启动命令 | 启动时间 | 技术栈 |
|------|------|---------|---------|--------|
| 后端 | 8690 | `cd casalyin-java && npm run dev` | ~30-40s | Spring Boot + Maven |
| 后台前端 | 5173 | `cd casalyin-server && npm run dev` | ~5s | React + Vite |
| C 端前台 | 3000 | `cd casalyin-Headless && npm run dev` | ~10s | Next.js |

---

## 重启服务

### 后端重启

```bash
# 方式一：使用 npm 脚本停止
cd casalyin-java
npm run dev:stop    # 停止占用 8690 端口的进程
npm run dev         # 重新启动

# 方式二：手动停止进程
# Windows PowerShell:
Get-NetTCPConnection -LocalPort 8690 | ForEach-Object { Stop-Process -Id $_.OwningProcess -Force }

# Git Bash:
netstat -ano | grep :8690 | awk '{print $5}' | xargs taskkill //PID //F
```

### 前端重启

```bash
# 停止前端服务（Ctrl+C 或关闭终端）
# 然后重新启动
cd casalyin-server
npm run dev
```

---

## 强制重新编译（后端）

当修改了 Java 代码但服务没有生效时：

```bash
cd casalyin-java

# 方式一：删除 target 目录（推荐）
rm -rf casalyin-admin/target
npm run dev

# 方式二：使用 Maven Wrapper 清理
./mvnw clean
npm run dev
```

---

## 常见问题

### 1. 端口被占用

**错误信息：** `Port 5173 is already in use` 或 `Port 8690 is already in use`

**解决方案：**
```bash
# 后端端口 8690
cd casalyin-java && npm run dev:stop

# 前端端口 5173
netstat -ano | grep :5173 | awk '{print $5}' | xargs taskkill //PID //F
```

### 2. 后端编译错误或 NoSuchMethodError

**原因：** 代码修改后没有重新编译

**解决方案：**
```bash
cd casalyin-java
rm -rf casalyin-admin/target    # 删除编译缓存
npm run dev                      # 重新编译并启动
```

### 3. Redis 缓存问题

**错误信息：** `ClassCastException: JSONArray cannot be cast to UserPermission`

**解决方案：**
```bash
redis-cli -n 1 flushdb
# 然后重新登录后台刷新权限缓存
```

### 4. Flyway 迁移失败

**错误信息：** `Validate failed: Migration checksum mismatch`

**原因：** 修改了已执行的 Flyway 迁移文件

**解决方案：**
- 开发环境：更新 `flyway_schema_history` 表的 checksum
- 生产环境：**绝对不能修改已执行的迁移文件**，必须新建 Vn+1 文件

---

## 环境要求

### 必需软件

- **Node.js**: v18+ （前端项目）
- **JDK**: 17.0.17（后端项目，路径：`C:\Program Files\Eclipse Adoptium\jdk-17.0.17.10-hotspot`）
- **Redis**: 本地运行（端口 6379）
- **MySQL**: 本地或远程数据库

### 环境变量

后端服务启动时会自动设置 `JAVA_HOME`，无需手动配置。

---

## 后台启动（可选）

如果需要在后台运行服务（不占用终端）：

```bash
# 使用 run_in_background 参数（Claude Code）
# 或使用 nohup（手动）
cd casalyin-java
nohup npm run dev > backend.log 2>&1 &

cd casalyin-server
nohup npm run dev > frontend.log 2>&1 &
```

---

## 测试账号

| 角色 | 入口 | 账号 | 密码 | 验证码 |
|------|------|------|------|--------|
| BASE | `/login` | `123456@qq.com` | `123456` | `123456` |
| COMPANY | `/login` | `123123@qq.com` | `123456` | `123456` |
| ADMIN | `/internal/auth-admin-portal` | `custom` | `123123` | 无 |

---

## 更新日志

- 2026-04-24: 创建启动指南，标准化启动流程
