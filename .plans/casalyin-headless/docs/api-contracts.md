# casalyin-headless - API 契约

> 前台调用的后端接口定义。字段名和类型的真理源头。
> 维护者：frontend-dev（API 变更时**必须**更新）
> 参考：docs/backend/api-conventions.md（共享规范）

## 通用约定

- 后端地址：`http://localhost:8690`（本地开发）
- 所有 POST 分页查询：`POST /{resource}/query`
- 详情：`GET /{resource}/detail/{id}`
- 路径全部 kebab-case，动态参数统一 `{id}`

## 响应格式

```json
{
  "code": 200,
  "msg": "success",
  "data": { ... }
}
```

## 各维度接口（随开发逐步补充）

### 产品（productos）

> TODO: 由 frontend-dev 开发时补充

### 品牌/企业（empresas）

> TODO: 由 frontend-dev 开发时补充

### 农艺师（agronomists）

> TODO: 由 frontend-dev 开发时补充

### 作物（cultivos）

> TODO: 由 frontend-dev 开发时补充

### 病虫害（plagas）

> TODO: 由 frontend-dev 开发时补充

### 化学成分（ingredientes）

> TODO: 由 frontend-dev 开发时补充
