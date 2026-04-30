# Bug 修复：产品文档上传后刷新丢失

## 问题描述
- URL: https://admin.casalyin.com/products/8001
- 现象：上传文件接口 `/api/product/document/upload` 返回成功，但刷新页面后文档列表为空
- 根因：后端 `ProductAddForm` 缺少 `documents` 字段，导致前端发送的文档数组被忽略

## 修复内容

### 1. ProductAddForm.java
**文件路径**: `casalyin-java/casalyin-admin/src/main/java/com/casalyin/admin/module/busniess/product/domain/form/ProductAddForm.java`

**修改**:
```java
import com.casalyin.admin.module.busniess.product.domain.dto.ProductDocumentDTO;

// 添加字段
@Schema(description = "产品文档列表")
private List<ProductDocumentDTO> documents;
```

### 2. ProductDraftService.java
**文件路径**: `casalyin-java/casalyin-admin/src/main/java/com/casalyin/admin/module/busniess/product/service/ProductDraftService.java`

**修改**: 在 `createDraftFromPublished()` 方法中添加 documents 序列化
```java
if (CollectionUtils.isNotEmpty(product.getDocuments())) {
    draft.setDocuments(JSON.toJSONString(product.getDocuments()));
}
```

## 部署状态
- ✅ 代码已提交: commit `01b21f8`
- ✅ 已推送到 GitHub: casalyin-java/main
- ✅ 服务器已更新代码
- ✅ backend 镜像已重新构建
- ✅ 容器已重启

## 需要 Review 的内容

### 1. 代码审查
请 Claude Team 审查以下修改是否完整和正确：
- ProductAddForm 是否需要添加 validation 注解？
- ProductDraftService 的序列化逻辑是否正确？
- 是否还有其他地方需要处理 documents 字段？

### 2. 功能测试
请测试以下场景：
1. **创建新产品并上传文档**
   - 访问 https://admin.casalyin.com/products/new
   - 填写产品信息
   - 上传文档（使用 /api/product/document/upload）
   - 保存草稿
   - 刷新页面，验证文档是否存在

2. **编辑已发布产品并上传文档**
   - 访问 https://admin.casalyin.com/products/8001
   - 上传新文档
   - 保存草稿
   - 刷新页面，验证文档是否存在

3. **提交审核和发布**
   - 创建带文档的产品草稿
   - 提交审核
   - 发布产品
   - 验证发布后的产品详情页是否显示文档

### 3. 数据验证
检查数据库中的数据是否正确：
```sql
-- 查看草稿表中的 documents 字段
SELECT id, product_name, documents 
FROM t_product_draft 
WHERE id = 8001;

-- 查看已发布产品表中的 documents 字段
SELECT id, product_name, documents 
FROM t_product 
WHERE id = 8001;
```

## 已知问题
服务器上有未提交的 OAuth2 和多语言功能代码（已 stash）：
```bash
# 查看 stash
cd /opt/casalyin/casalyin-java
git stash list

# 如果需要恢复
git stash pop
```

## 相关文件
- ProductDocumentDTO.java (已存在，无需修改)
- ProductAddForm.java (已修改)
- ProductUpdateForm.java (已有 documents 字段，无需修改)
- ProductDraftService.java (已修改)
- ProductEditor.tsx (前端，无需修改，已兼容两种格式)

## 下一步
1. Claude Team review 代码
2. 执行完整的功能测试
3. 如果发现问题，反馈给 Hermes
4. 如果测试通过，关闭此任务
