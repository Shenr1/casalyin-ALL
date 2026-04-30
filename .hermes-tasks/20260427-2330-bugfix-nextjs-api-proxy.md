# 任务：修复 Next.js API 代理配置

**状态：** DONE  
**创建时间：** 2026-04-27 23:30  
**优先级：** 🔴 HIGH（生产环境故障）  
**完成时间：** 2026-04-27 23:50

---

## 问题描述

生产环境 Next.js（casalyin-headless）无法调用后端 API，所有请求返回 500 错误。

**错误日志：**
```
产品筛选 API 代理错误: TypeError: fetch failed
Error: connect ECONNREFUSED 47.77.200.69:443
```

**用户访问：** http://47.77.200.69:3000/api/product/filter  
**实际错误：** Next.js 尝试通过 HTTPS 443 端口连接后端，但服务器未配置 HTTPS

---

## 根因分析

**这是典型的 CI/CD 环境配置缺失问题！**

1. **本地开发环境**：Next.js 使用 `localhost:1025` 或开发环境配置，工作正常
2. **生产环境**：代码中可能硬编码了公网地址 `https://47.77.200.69:443`
3. **问题**：Next.js 在 Docker 容器内做 SSR 时，无法通过公网地址访问同一服务器上的后端
4. **正解**：服务端应该用 Docker 内部地址 `http://casalyin-backend-1:1025`

**CI/CD 缺失的环节：**
- 没有区分开发/生产环境的 API 配置
- 没有使用环境变量来动态配置 API 地址
- Docker 构建时没有注入正确的环境变量
- 缺少环境配置文件（.env.production）

---

## 修改方案

### 1. 检查 Next.js 代码中的 API 调用配置

查找文件（可能的位置）：
- `casalyin-Headless/lib/api/client.ts`
- `casalyin-Headless/lib/config.ts`
- `casalyin-Headless/next.config.js`

找到 API base URL 的配置代码，应该类似：
```typescript
const API_BASE_URL = process.env.NEXT_PUBLIC_API_URL || 'https://47.77.200.69'
```

### 2. 修改为支持环境变量

```typescript
// 服务端使用内部地址，客户端使用公网地址
const getApiBaseUrl = () => {
  if (typeof window === 'undefined') {
    // 服务端（SSR）：使用 Docker 内部地址
    return process.env.API_INTERNAL_URL || 'http://casalyin-backend-1:1025'
  }
  // 客户端：使用公网地址
  return process.env.NEXT_PUBLIC_API_URL || 'http://47.77.200.69:1025'
}

export const API_BASE_URL = getApiBaseUrl()
```

### 3. 更新 Dockerfile 或 docker-compose.yml

在 `casalyin-Headless/deploy/docker-compose.yml` 中添加环境变量：

```yaml
services:
  casalyin-headless:
    environment:
      - API_INTERNAL_URL=http://casalyin-backend-1:1025
      - NEXT_PUBLIC_API_URL=http://47.77.200.69:1025
```

### 4. 如果使用了 Next.js rewrites

检查 `next.config.js` 中是否有 `rewrites` 配置：

```javascript
async rewrites() {
  return [
    {
      source: '/api/:path*',
      destination: process.env.API_INTERNAL_URL + '/api/:path*'
    }
  ]
}
```

---

## 验证方式

1. 重新构建并部署 Next.js 容器
2. 检查日志：`docker logs casalyin-headless-1 --tail 50`
3. 测试 API：`curl http://47.77.200.69:3000/api/product/filter`
4. 应该返回正常的 JSON 数据，而不是 500 错误

---

## 执行记录

- [x] 定位 API 配置代码
- [x] 修改为环境变量驱动
- [x] 更新 Docker 配置
- [x] 提交代码
- [ ] 通知 Hermes 验证部署

**执行时间：** 2026-04-28 00:12  
**执行者：** Claude CLI  

**完成的修改：**
1. 创建了 `lib/api-config.ts` - 实现服务端/客户端 API 地址自动选择
2. 批量更新了 24 个 API Route 文件，使用 `getApiBaseUrl()` 替代硬编码配置
3. 更新了 `next.config.js` 的环境变量配置
4. 更新了 `deploy/docker-compose.yml` 和 `deploy/docker-compose.prod.yml`，添加环境变量：
   - `API_INTERNAL_URL=http://backend:1025` (服务端使用)
   - `NEXT_PUBLIC_API_URL=http://47.77.200.69:1025` (客户端使用)

**Git Commit：** 0cb6306

---

## 相关信息

**后端服务：**
- 容器名：casalyin-backend-1
- 内部地址：http://casalyin-backend-1:1025
- 外部地址：http://47.77.200.69:1025
- Context Path：/api

**Next.js 服务：**
- 容器名：casalyin-headless-1
- 端口：3000
- 网络：casalyin_default（与后端在同一网络）

**测试命令：**
```bash
# 容器内部测试（应该成功）
docker exec casalyin-headless-1 wget -O- http://casalyin-backend-1:1025/api/product/filter

# 外部测试（修复后应该成功）
curl http://47.77.200.69:3000/api/product/filter
```
