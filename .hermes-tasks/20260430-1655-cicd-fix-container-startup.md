# CI/CD 修复：构建后自动启动容器

## 问题描述
当前 CI/CD 脚本在构建镜像后不会自动启动容器，导致每次部署后需要手动执行 `docker compose up -d`。

## 根本原因
三个子项目的 `.github/workflows/deploy.yml` 都存在同样的问题：

1. **第 47-52 行**：执行 `docker compose build` 构建镜像
2. **第 50-57 行**：构建完后又执行了一遍 `docker compose down` 和容器清理
3. **第 58 行**：执行 `docker compose up -d --force-recreate`

**问题点**：第 50-57 行的清理操作是**重复的**（第 40-46 行已经清理过一次），而且在构建后再次清理可能导致：
- 网络被删除后 `up -d` 时需要重建
- 容器名冲突
- 资源竞争

## 修改方案

### 方案 A：简化清理逻辑（推荐）
删除第 50-57 行的重复清理，保留第 40-46 行的预清理 + 第 58 行的启动：

```yaml
# 构建并重启
cd deploy
# 预清理（只执行一次）
docker compose down --remove-orphans 2>/dev/null || true
for id in $(docker ps -aq -f label=com.docker.compose.project=casalyin); do [ -n "$id" ] && docker rm -f "$id"; done
for id in $(docker ps -aq -f label=com.docker.compose.project=deploy); do [ -n "$id" ] && docker rm -f "$id"; done
docker rm -f deploy-headless-1 deploy-server-1 deploy-backend-1 casalyin-headless casalyin-server casalyin-backend casalyin-headless-1 casalyin-server-1 casalyin-backend-1 2>/dev/null || true
docker network rm deploy_default 2>/dev/null || true
docker network rm casalyin_default 2>/dev/null || true
docker builder prune -f || true

# 构建镜像
docker compose build headless
docker compose build server
docker compose build backend

# 启动容器
docker compose up -d --force-recreate

# 清理旧镜像
docker image prune -f

echo "部署完成！"
```

### 方案 B：添加启动验证
在 `docker compose up -d` 后添加健康检查：

```bash
docker compose up -d --force-recreate

# 等待容器启动
sleep 10

# 验证容器状态
if ! docker compose ps | grep -q "Up"; then
  echo "容器启动失败！"
  docker compose logs --tail=50
  exit 1
fi

echo "部署完成，所有容器已启动！"
```

## 需要修改的文件
1. `casalyin-java/.github/workflows/deploy.yml`（第 50-57 行）
2. `casalyin-server/.github/workflows/deploy.yml`（第 53-60 行）
3. `casalyin-Headless/.github/workflows/deploy.yml`（第 50-57 行）

## 验证方式
1. 修改后推送代码触发 CI/CD
2. 等待 3-5 分钟 CI/CD 完成
3. SSH 到服务器执行：`docker compose ps`
4. 确认所有容器状态为 `Up`
5. 访问 https://app.casalyin.com 和 https://admin.casalyin.com 验证服务正常

## 执行记录
- [x] 修改 casalyin-java/.github/workflows/deploy.yml
- [x] 修改 casalyin-server/.github/workflows/deploy.yml
- [x] 修改 casalyin-Headless/.github/workflows/deploy.yml
- [ ] 提交并推送
- [ ] 验证 CI/CD 执行结果
- [ ] 验证容器自动启动
- [ ] 更新 CLAUDE.md 中的 CI/CD 说明

## 状态
IN_PROGRESS - backend-dev 已完成修改，等待 reviewer 审查

---
创建时间：2026-04-30 16:55
创建者：Hermes
