# 后端接口规范

> 来源：backend_api.md
> 最后更新：2026-04-11

---

## 技术栈

Sa-Token + MyBatis Plus，权限控制通过 `@SaCheckPermission` + 服务层 `UserTypeEnum` 检查实现。

- `UserTypeEnum` 只有 `USER` 和 `ADMIN` 两种，**没有 COMPANY**
- ADMIN 角色直接拥有全量权限码，不走角色权限关联表
- 权限缓存在 Redis，角色/权限变更时清除缓存

---

## HTTP 方法规范

- **删除操作统一用 `@DeleteMapping`**，禁止用 `@GetMapping` 做删除
- **上下架/状态切换统一用 `@PostMapping`**，禁止用 `@PutMapping`
- **查询用 `@GetMapping`（无 body）或 `@PostMapping`（有查询条件 body）**
- 同一接口禁止同时注册多个 HTTP 方法

---

## 路径命名规范

### 全局规则

- 路径全部 **kebab-case**，禁止下划线（`query_overview` → `query-overview`）
- 动态参数统一用 **`{id}`**，不带业务前缀（禁止 `{storeId}`、`{productId}`、`{brandId}`）
- 上下架动词在后：`/{resource}/offline/{id}`、`/{resource}/online/{id}`（禁止 `/{id}/offline`）

### 主表操作标准路径

```
POST   /{resource}/create          # 新建（统一用 create，禁止 add）
POST   /{resource}/update          # 更新
DELETE /{resource}/delete/{id}     # 删除单条
DELETE /{resource}/batch-delete    # 批量删除（kebab-case，禁止 batchDelete）
POST   /{resource}/query           # 分页查询（有 body）
GET    /{resource}/detail/{id}     # 详情
POST   /{resource}/online/{id}     # 上架
POST   /{resource}/offline/{id}    # 下架
```

### Admin 专属标准路径

```
/admin/{resource}/create           # Admin 直接建主表
/admin/{resource}/update           # Admin 直接更新
/admin/{resource}/delete/{id}      # Admin 删除（DELETE 方法）
/admin/{resource}/batch-delete     # Admin 批量删除（DELETE 方法）
/admin/{resource}/query            # Admin 查询
/admin/{resource}/certify/{id}     # 认证状态切换
/admin/{resource}/offline/{id}     # Admin 强制下架
```

### 草稿标准路径

```
POST   /{resource}/draft/create              # 新建草稿
POST   /{resource}/draft/update/{id}         # 更新草稿（统一一个接口，禁止拆 update-content）
POST   /{resource}/draft/submit/{id}         # 提交审核
POST   /{resource}/draft/cancel/{id}         # 撤回审核
DELETE /{resource}/draft/delete/{id}         # 删除草稿
GET    /{resource}/draft/detail/{id}         # 草稿详情
GET    /{resource}/draft/my                  # 我的草稿列表
GET    /{resource}/draft/all                 # 所有草稿（Admin）
GET    /{resource}/draft/pending             # 待审核列表（Admin）
GET    /{resource}/draft/rejected            # 已拒绝列表（Admin）
POST   /{resource}/audit/approve/draft/{id} # 审核通过
POST   /{resource}/audit/reject/draft/{id}  # 审核拒绝
```

### 公开接口标准路径

```
/{resource}/public/query           # 公开分页查询
/{resource}/public/detail/{id}     # 公开详情
```

禁止用 `/public/{resource}/...` 前置格式。禁止路径含 `/admin/` 却同时标注 `@NoNeedLogin`。

---

## 鉴权注解规范

- 需要登录但无特定权限码的接口：**不加任何注解**（框架全局拦截器保证登录态）
- 无需登录的公开接口：**统一用 `@NoNeedLogin`**，禁止用 `@SaIgnore`（后者完全绕过 Sa-Token 所有鉴权）
- 查询接口不得要求写权限码

---

## 权限码命名规范

- 草稿增删改查、上下架：统一用 `{resource}:edit`（如 `store:edit`、`product:edit`）
- 主表 CRUD：`{resource}:create`、`{resource}:update`、`{resource}:delete`、`{resource}:query`
- Admin 审核操作：`{resource}:audit`

---

## 四维度接口完整性矩阵（已落地，2026-04-03）

| 接口 | 店铺 | 产品 | 农艺师 | 品牌 |
|------|------|------|--------|------|
| 主表 create（Admin）| ✅ `POST /admin/store/create` | ✅ `POST /admin/product/create` | ✅ `POST /admin/agronomist/create` | ✅ `POST /admin/brand/create` |
| online/offline（用户）| ✅ `/store/online/{id}` `/offline/{id}` | ✅ `/product/online/{id}` `/offline/{id}` | ✅ `/agronomist/online/{id}` `/offline/{id}` | — 品牌无上下架 |
| admin/offline（强制）| ✅ `POST /admin/store/offline/{id}` | ✅ `POST /admin/product/offline/{id}` | — | — |
| admin/certify | ✅ `POST /admin/store/certify/{id}` | ✅ `POST /admin/product/certify/{id}` | ✅ `POST /admin/agronomist/toggle-certified/{id}` | — 品牌无认证状态 |
| draft/create | ✅ | ✅ | ✅ | ✅ |
| draft/submit、cancel、delete、detail、my | ✅ | ✅ | ✅ | ✅ |
| draft/all、pending、rejected | ✅ | ✅ | ✅ | ✅ |
| audit/approve、reject | ✅ | ✅ | ✅ | ✅ |
| public/query、detail | ✅ | ✅ | ✅ | ✅ |

**特殊说明：**
- 农艺师保留 `POST /agronomist/my-profile` 和 `POST /agronomist/my-expertise` 直接更新主表（不走草稿），有意设计，降低用户创建门槛
- 品牌 `draft/create`：body 中有 `brandId` = 修改已发布品牌；无 `brandId` = 新建品牌
- 品牌无 online/offline、无 certify

---

## 接口去重规范

- 同一功能禁止绑定多个路径
- 废弃的旧路由直接删除，开发阶段不保留兼容别名

---

## Dashboard Admin 接口原则

**规则：** 新增 Dashboard 模块前问「这个数字会让 Admin 点进去做什么？」，不能回答则不加。

已有的 count 专用接口：
- `GET /brand/draft/pending/count` — 需 `brand:audit` 权限
- `GET /brand/apply/pending/count` — 需 `brand:audit` 权限
