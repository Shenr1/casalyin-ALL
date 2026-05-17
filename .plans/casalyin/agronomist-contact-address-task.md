# 农艺师添加联系方式和地址字段任务

## 需求背景
后台农艺师详情页面缺少联系方式和地址信息，需要参考店铺的字段结构添加这两个重要字段。

## 字段设计（参考店铺）

### 1. 联系方式字段
- **字段名**：`contacts`
- **类型**：`text`（JSON 格式）
- **格式**：`[{name?, email?, phone?}]`
- **说明**：联系人列表，支持多个联系人，每个联系人可包含姓名、邮箱、电话
- **参考**：店铺的 `contacts` 字段

### 2. 地址字段
- **字段名**：`cities`
- **类型**：`json`
- **格式**：`["城市1", "城市2", ...]`
- **说明**：农艺师服务的城市列表（多选），不需要精确到街道地址
- **原因**：农艺师可能服务多个城市，只需要城市级别即可

## 涉及范围

### 1. 数据库层面
- `t_agronomist` 表：添加 `contacts` 和 `cities` 字段
- `t_agronomist_draft` 表：添加 `contacts` 和 `cities` 字段

### 2. 后端层面
- `AgronomistEntity.java`：添加字段
- `AgronomistDraftEntity.java`：添加字段
- `AgronomistVO.java`：添加字段
- `AgronomistDetailVO.java`：添加字段
- `AgronomistDraftVO.java`：添加字段
- `AgronomistAddForm.java`：添加字段
- `AgronomistUpdateForm.java`：添加字段
- `AgronomistDraftAddForm.java`：添加字段
- `AgronomistDraftUpdateForm.java`：添加字段
- Service 层：处理 JSON 字段的序列化/反序列化

### 3. 前端后台层面
- `AgronomistEditor.tsx`：添加联系方式和城市选择表单项
- 参考 `StoreEditor.tsx` 的联系方式实现
- 使用 `PlacesCityAutocomplete` 组件实现城市多选

### 4. 前端 C 端层面
- `casalyin-Headless/app/agronomists/[id]/page.tsx`：展示联系方式和服务城市
- 参考产品详情页的联系方式弹窗实现

## 实施步骤

### Step 1: backend-dev - 数据库迁移
创建 Flyway 迁移文件，添加字段：
```sql
ALTER TABLE t_agronomist 
ADD COLUMN contacts TEXT COMMENT '联系人列表JSON:[{name?,email?,phone?}]',
ADD COLUMN cities JSON COMMENT '服务城市列表';

ALTER TABLE t_agronomist_draft 
ADD COLUMN contacts TEXT COMMENT '联系人列表JSON:[{name?,email?,phone?}]',
ADD COLUMN cities JSON COMMENT '服务城市列表';
```

### Step 2: backend-dev - 后端代码
1. 更新 Entity 类
2. 更新 VO 类
3. 更新 Form 类
4. 更新 Service 层（如需要）

### Step 3: frontend-dev - 后台管理页面
1. 在 `AgronomistEditor.tsx` 中添加联系方式表单项
   - 参考 `StoreEditor.tsx` 的实现
   - 支持添加/删除多个联系人
   - 每个联系人包含：姓名、邮箱、电话
2. 添加城市选择表单项
   - 使用 `PlacesCityAutocomplete` 组件
   - 支持多选城市

### Step 4: frontend-dev - C 端详情页
1. 在农艺师详情页展示联系方式
   - 参考产品详情页的联系方式弹窗
   - 显示电话和邮箱
2. 展示服务城市列表
   - 使用 Badge 组件展示

## 验收标准
1. 数据库字段添加成功，Flyway 迁移无报错
2. 后台管理页面可以正常添加/编辑联系方式和城市
3. 后台详情页可以正常展示联系方式和城市
4. C 端详情页可以正常展示联系方式和城市
5. 草稿流程正常（创建草稿、提交审核、审核通过后同步到正式表）

## 注意事项
1. 联系方式字段为 JSON 格式，需要在前后端正确处理序列化/反序列化
2. 城市字段使用 MySQL JSON 类型，需要使用 JSON 函数查询
3. 参考店铺的实现，保持代码风格一致
4. 前端表单验证：至少填写一个联系方式（邮箱或电话）
5. C 端展示时需要处理空值情况
