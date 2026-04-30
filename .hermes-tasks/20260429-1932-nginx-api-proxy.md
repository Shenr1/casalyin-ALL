# 后台登录问题修复 - Nginx API 反向代理缺失

**状态：** DONE  
**创建时间：** 2026-04-29 19:32  
**完成时间：** 2026-04-29 19:40  
**优先级：** P0（生产环境故障）

---

## 问题描述

生产环境后台管理系统（http://47.77.200.69:8080）无法登录，报 "network error"。

**根因：** `casalyin-server/nginx.conf` 缺少 API 反向代理配置，导致前端请求 `/api/*` 时被当作静态文件处理，后端报错 `No static resource`。

---

## 已完成修改

✅ 修改 `casalyin-server/nginx.conf`，添加 API 反向代理：

```nginx
location /api/ {
    proxy_pass http://casalyin-backend-1:1025;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
}
```

---

## 待执行部署步骤

需要在生产服务器（47.77.200.69）执行：

```bash
# 1. 进入部署目录
cd /opt/casalyin/casalyin-server/

# 2. 拉取最新代码（包含修复后的 nginx.conf）
git pull origin main

# 3. 重新构建镜像
docker-compose build server

# 4. 重启容器
docker-compose up -d --force-recreate server

# 5. 验证
curl -I http://localhost:8080/api/health
# 应该返回 200 OK，而不是 404
```

---

## 验证方式

1. 访问 http://47.77.200.69:8080
2. 输入账号密码登录
3. 应该能正常进入后台管理系统

---

## 执行记录

- [x] 代码已提交到 main 分支（3 次提交）
- [x] 生产服务器已拉取最新代码
- [x] 镜像已重新构建
- [x] 容器已重启
- [x] 登录功能已验证通过

**最终方案：**
使用 Nginx resolver + 变量方式动态解析后端主机名，避免启动时解析失败：
```nginx
location /api/ {
    resolver 127.0.0.11 valid=30s;
    set $backend backend:1025;
    proxy_pass http://$backend;
    ...
}
```

**验证结果：**
- http://47.77.200.69:8080/ → 200 OK（前端页面）
- http://47.77.200.69:8080/api/health → 200 OK（API 代理成功）

---

## 备注

- 此问题影响所有后台 API 请求，不仅是登录
- 前台 C 端（3000端口）不受影响，因为它有独立的 API 配置
- 修复后需要清除浏览器缓存再测试
