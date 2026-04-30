# 国际化混合处理方案

**创建时间**: 2026-05-01 00:30  
**状态**: ✅ 已完成并合并到 main  
**负责**: Claude CLI Team  
**分支**: feature/i18n-extract-hardcoded（已合并删除）

## 背景

用户要求对 http://localhost:5173/vip/my 页面及整个前端项目进行国际化，包括所有硬编码的中文文案。

## 方案

采用**混合处理**：
- **简单文案**（纯文本）→ 自动化脚本提取
- **复杂文案**（带变量插值、嵌套元素）→ 手动处理

## 已完成

1. ✅ 改进 i18n 提取脚本 `scripts/extract-i18n.cjs`
   - 跳过包含变量插值的复杂模板
   - 跳过嵌套元素的文本
   - 跳过对象字面量的值（如 BRAND_MAP）
   - 合并而非覆盖现有翻译文件

2. ✅ 提交脚本改进到分支 `feature/i18n-extract-hardcoded`

## 待处理

### 1. 运行自动化脚本（简单文案）

```bash
cd ~/Documents/Project/casalyin-ALL/casalyin-server
node scripts/extract-i18n.cjs --execute
```

预计提取：
- 7 个文件
- 54 处简单文案
- 63 条新翻译 key

### 2. 手动处理复杂文案

脚本会跳过以下类型的文案，需要手动处理：

#### 2.1 VipBanner.tsx
```tsx
// 原代码
<Typography.Text>
  当前{dimensionLabel}为<Typography.Text strong>普通版</Typography.Text>，升级 VIP 后可{benefitDesc}
</Typography.Text>

// 改为
<Typography.Text>
  {t('vipBanner.upgradePrompt', { 
    dimension: dimensionLabel, 
    benefit: benefitDesc 
  })}
</Typography.Text>

// zh-CN.json 添加
"vipBanner": {
  "upgradePrompt": "当前{{dimension}}为普通版，升级 VIP 后可{{benefit}}",
  "expireTime": "到期时间：",
  "renew": "续费",
  "upgrade": "立即升级"
}
```

#### 2.2 CompositionSelect.tsx
检查是否有类似的复杂模板。

#### 2.3 BrandSelect.tsx
检查是否有类似的复杂模板。

#### 2.4 DashboardExample.tsx
检查是否有类似的复杂模板。

### 3. 处理用户提供的文案列表

用户提到的文案（可能在 /vip/my 页面）：
- 店铺名称
- 省份/州
- 城市
- 街道地址
- 联系人信息
- 店铺描述
- 添加联系人
- 产品名称
- 品牌
- 状态
- 未知状态
- 选择已有成分
- 产品用途管理
- 害虫
- 农作物
- 使用剂量
- PC值
- LM值

**操作**：
1. 定位这些文案在源代码中的位置
2. 检查是否已被脚本处理
3. 如果未处理，手动提取到 zh-CN.json

### 4. 补充英文和西班牙语翻译

脚本只生成中文翻译，需要补充：
- `src/locales/en.json`
- `src/locales/es-ES.json`

可以使用翻译工具或 AI 辅助。

### 5. 本地测试

```bash
# 启动前端
cd ~/Documents/Project/casalyin-ALL/casalyin-server
npm run dev

# 启动后端（如果需要）
cd ~/Documents/Project/casalyin-ALL/casalyin-java/casalyin-admin
export JAVA_HOME=/opt/homebrew/Cellar/openjdk@17/17.0.19/libexec/openjdk.jdk/Contents/Home
../mvnw spring-boot:run -Dspring-boot.run.profiles=dev -Dspring.devtools.restart.enabled=false
```

测试：
- 切换语言（中文/英文/西班牙语）
- 检查所有页面是否正常显示
- 检查是否有遗漏的硬编码文案

### 6. 提交和推送

```bash
git add .
git commit -m "feat: 完成国际化提取 - 混合自动化和手动处理"
git push origin feature/i18n-extract-hardcoded
```

## 脚本限制

`scripts/extract-i18n.cjs` 的限制：
1. **跳过复杂模板** - 包含变量插值或嵌套元素的文本
2. **跳过对象字面量** - 如 `BRAND_MAP = { 1: '拜耳' }`
3. **生成 Unicode 转义** - key 中的中文会被转义（如 `\u5230\u671F`）
4. **不处理模板字符串** - 如 `` `当前${x}` ``
5. **不处理 JSX 属性中的复杂表达式**

## 注意事项

1. **不要覆盖现有翻译** - 脚本已修复，会合并而非覆盖
2. **保持语义完整** - 手动处理时，保持完整的句子而非碎片化 key
3. **使用插值变量** - 对于动态内容，使用 `{{variable}}` 而非拆分文本
4. **测试充分** - 确保所有语言切换正常

## 参考

- i18next 文档: https://www.i18next.com/
- react-i18next 文档: https://react.i18next.com/
- 项目国际化指南: `I18N_EXTRACTION_GUIDE.md`
