# B1 后端：产品维度 submitDraft 敏感词检测

> 状态: ✅ 已完成（代码已就绪，无需修改）
> 日期: 2026-04-12

## 任务描述

检查 `ProductDraftService.submitDraft()` 是否缺少敏感词检测逻辑。

## 检查结论

**代码已完整实现，无需任何修改。**

### 代码位置

`ProductDraftService.java` 第 578-591 行（`submitDraft()` 方法内）：

```java
// 违规词检测（info/description 可能是富文本，先提取纯文本再检测）
String plainInfo = draftEntity.getInfo() != null ? Jsoup.parse(draftEntity.getInfo()).text() : null;
String plainDesc = draftEntity.getDescription() != null ? Jsoup.parse(draftEntity.getDescription()).text() : null;
Map<String, BlockLevelEnum> hits = sensitiveWordService.checkAll(
        draftEntity.getProductName(), draftEntity.getSku(), plainInfo, plainDesc);

if (!hits.isEmpty()) {
    boolean hasHard = hits.values().contains(BlockLevelEnum.HARD);
    if (hasHard) {
        String hitWords = String.join(", ", hits.keySet());
        return ResponseDTO.userErrorParam("内容含有违规词: " + hitWords + "，请修改后重新提交");
    }
    // SOFT: COMPANY 免审，跳过 SOFT 词检测；BASE 走人工审核
}
```

### 四维度敏感词检测一致性对比

| 维度 | 检测字段 | 状态 |
|------|---------|------|
| 店铺 | storeName, plainInfo | ✅ |
| 品牌 | brandName, brandCode, description | ✅ |
| 农艺师 | name, title, organization, plainProfile | ✅ |
| 产品 | productName, sku, plainInfo, plainDesc | ✅ |

所有维度逻辑一致：
- HARD 词 → 直接报错
- SOFT 词 → COMPANY 免审直发，BASE 转 PENDING 人工审核

## 变更记录

无代码变更。无 DB 变更。
