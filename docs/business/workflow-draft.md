# 草稿审核流程与状态规则

> 来源：business_workflow.md
> 最后更新：2026-04-11

---

## 状态流转

```
创建 → [DRAFT 草稿] → 用户提交 → [PENDING 待审核] → admin 通过 → 写入主表（已发布）
                                                     ↘ admin 拒绝 → [REJECTED 已拒绝] → 用户可修改重新提交
```

## draftStatus 枚举值

| 值 | 枚举 | 含义 |
|----|------|------|
| 0 | DRAFT | 草稿，已保存未提交 |
| 1 | PENDING | 待审核，已提交等待 admin 操作 |
| 2 | REJECTED | 已拒绝，可继续修改再提交 |

- 审核通过（APPROVED）后写入主表，草稿表只存 0/1/2
- ADMIN 操作无需走草稿审核流程
- 注释统一写法：`// 0-草稿, 1-待审核, 2-已拒绝`

---

## 各维度草稿表

| 维度 | 草稿表 |
|------|--------|
| 产品 | t_product_draft |
| 品牌 | t_brand_draft |
| 店铺 | t_store_draft |
| 农艺师 | t_agronomist_draft |

---

## 草稿删除规则

- BASE/COMPANY 只能删除 `draftStatus=0`（未提交草稿）
- PENDING 不可删：已进入审批流，存在 Admin 参与，不允许单方面删除
- REJECTED 不可删：已有 Admin 操作记录，保留以供用户修改后重新提交
- ADMIN 可删除任意状态的草稿

---

## 品牌 / 店铺 / 农艺师：免审直发规则

**背景：** 三个维度的内容属于用户唯一专属，高频修改走全审核流程阻碍用户意愿。产品维度为共用，保持原有审核流程不变。

### 提交时的违规词检测分支

```
submitDraft()
  ├─ 命中 HARD 词 → 返回错误提示，草稿保留，用户自行修改
  ├─ 命中 SOFT 词 → COMPANY 角色例外，命中 SOFT 词也直接发布；其他角色 fall through 原有 PENDING 流程（转人工审核）
  └─ 无命中      → 免审直发（写主表 + 记内容变更日志 + 删草稿）
```

### 各维度检测字段

| 维度 | 检测字段 |
|------|---------|
| 产品 | productName, sku, info（Jsoup 提取纯文本）, description（Jsoup 提取纯文本）|
| 品牌 | brandName, brandCode, description |
| 店铺 | storeName, info（Jsoup 提取纯文本）|
| 农艺师 | name, title, organization, profile（Jsoup 提取纯文本）|

富文本字段（info / profile）必须先用 `Jsoup.parse(html).text()` 提取纯文本再送检。

### 违规词库设计

- 表：`t_sensitive_word`，字段：word, block_level（HARD/SOFT）, category
- 使用纯 Java DFA（Trie 树）实现，不引入三方依赖
- `SensitiveWordService` 在 `@PostConstruct` 时从 DB 加载，每次写操作后 `reload()`（volatile swap，线程安全）
- 后台管理接口权限：`sensitiveWord:manage`（ADMIN Only）

### 内容变更日志

- 表：`t_content_change_log`，记录免审直发的变更
- diff_json 结构：`[{field, label, before, after}]`，新建时 before 全为 null
- `ContentChangeLogService.record()` 无 `@Transactional`，由调用方（DraftService）的事务保护
- 查询权限：`contentLog:query`（ADMIN Only）
- 前台入口：Admin 菜单 `/content-logs`

### 直发时 actionType 处理

| actionType | 含义 | 处理 |
|-----------|------|------|
| 1 | 新建 | INSERT 主表，before 字段全为 null |
| 2 | 更新 | 先 SELECT 主表旧值，构建 diffJson，再 UPDATE |

> 农艺师特殊：actionType=1 时若主表已存在同 userId 记录（uk_user_id 约束），改为 UPDATE 而非 INSERT。

---

## 前端按钮文案规范

品牌/店铺/农艺师三个维度的提交按钮，**不再使用"提交审核"文案**，统一改为"发布"语义：

| 场景 | 按钮文案 |
|------|---------| 
| 新建页首次发布 | "发布品牌" / "发布店铺" / "发布农艺师" |
| 草稿编辑页发布 | "发布" |
| 被拒绝/下架后重新发布 | "重新发布" |
| 成功提示 toast | "品牌已发布" / "店铺已发布" / "农艺师已发布" |

**保留不变的按钮：**
- "保存草稿" — 用户仍可先保存不发布
- "撤回审核" / "取消审核" — SOFT 词命中后草稿进入 PENDING，用户仍需要能撤回
- Admin 侧"审核通过" / "审核拒绝" — SOFT 词命中的草稿走人工审核

---

## 产品归属逻辑

- BASE 用户创建了关联某品牌的产品，产品归属于该 BASE 用户
- 该品牌有了 COMPANY 账号后，触发"同步产品"，产品 userId 更新为 COMPANY 账号，原 BASE 用户失去 CRUD 权限
