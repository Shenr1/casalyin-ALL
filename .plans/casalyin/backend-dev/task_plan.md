# backend-dev - 任务计划

> 角色: 后端开发（Spring Boot + Sa-Token + MyBatis Plus + Flyway）
> 状态: standby
> 当前版本: v1.4 待启动（等 Task #4 前端收尾后由 team-lead 通知）

## 已完成任务

| 任务 | 状态 |
|------|------|
| Bug 13: POST /brand/apply/submit → 10001 | ✅ CacheConfig FastJSON→Jackson 修复，v1.2 已验证 |
| Bug 14: POST /product/sync-creator → 10001 | ✅ Bug 12 权限码修复已覆盖，用户确认返回 code:0 "共更新9个产品" |
| GET /brand/my-products 接口 | ✅ 完成（BrandProductVO + SQL + Service + Controller），e2e 联调 10/10 PASS，v1.3 正式关闭 |

## v1.4 — 平台认证申请体系（待启动）

> 等 team-lead 通知后开始

- [ ] 期一：Controller 层暴露 HTTP 接口（Service 已写好）+ Flyway 建表确认

## 项目信息

工作目录：casalyin-java/
规范文档：docs/backend/（api-conventions, database）
业务规范：docs/business/（workflow-draft, online-offline, roles-permissions）

## 关键约束

- 接口路径必须 kebab-case，动态参数统一 {id}
- 上下架必须用 POST，禁止 GET/PUT
- 所有 DB 变更必须写 Flyway 文件
- 草稿 PENDING 状态后端拦截所有写操作

## 备注

- 每次修复后通知 e2e-tester 运行相关测试
- 重大变更请求 reviewer 审查
