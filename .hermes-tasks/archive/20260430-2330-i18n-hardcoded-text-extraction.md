# 国际化硬编码文案提取任务

**创建时间**: 2026-04-30 23:30  
**状态**: ✅ 已完成并合并到 main  
**负责**: Claude CLI Team  
**分支**: feature/i18n-extract-hardcoded（已合并删除）

## 任务目标
将 casalyin-server 前端项目中所有硬编码的中文文案提取到国际化文件中。

## 现状分析

### 项目结构
- 国际化框架：i18next + react-i18next
- 语言文件：`src/locales/zh-CN.json`, `en.json`, `es-ES.json`
- 配置文件：`src/i18n.ts`

### 硬编码文案统计
- **86 个文件**包含硬编码中文
- **Top 5 重灾区**：
  1. `pages/ProductEditor.tsx` - 151 处
  2. `pages/SystemRoles.tsx` - 140 处
  3. `pages/ProductList.tsx` - 118 处
  4. `api/endpoints.ts` - 115 处（注释）
  5. `api/brand.ts` - 107 处（注释）

### 文案类型分类
1. **UI 文案**：按钮、标签、提示信息
2. **表单文案**：placeholder、label、验证消息
3. **注释**：代码注释（不需要国际化）
4. **日志/调试信息**：console.log（不需要国际化）

## 技术方案

### 工具选择

#### 方案 A：自动化脚本（推荐）
使用 AST 解析 + 规则匹配自动提取和替换：

**工具栈**：
- `@babel/parser` - 解析 TypeScript/JSX
- `@babel/traverse` - 遍历 AST
- `@babel/generator` - 生成代码
- `recast` - 保留代码格式

**优势**：
- 精确识别 JSX 文本节点、字符串字面量
- 自动生成 i18n key
- 保留代码格式和注释
- 可批量处理

**劣势**：
- 需要编写脚本（约 2-3 小时）
- 可能误判边界情况

#### 方案 B：i18next-scanner（半自动）
使用官方扫描工具：

```bash
npm install -D i18next-scanner
```

**优势**：
- 官方工具，成熟稳定
- 配置简单

**劣势**：
- 只能扫描已使用 `t()` 的文案
- 不能自动替换硬编码文案

#### 方案 C：人工 + IDE 辅助
使用正则搜索 + 手动替换：

**优势**：
- 精确控制
- 适合小规模

**劣势**：
- 耗时长（预计 8-10 小时）
- 容易遗漏
- 易出错

### 推荐方案：混合方案

**阶段 1：自动化提取（80%）**
1. 编写 AST 脚本自动提取 JSX 文本节点和模板字符串中的中文
2. 自动生成 i18n key（基于文件路径 + 上下文）
3. 自动替换为 `t('key')`
4. 更新 `zh-CN.json`

**阶段 2：人工审查（20%）**
1. 检查自动生成的 key 是否语义化
2. 处理动态文案（插值）
3. 处理复杂场景（条件渲染、循环）
4. 补充英文和西班牙语翻译

## 实施计划

### Step 1: 创建独立分支
```bash
cd ~/Documents/Project/casalyin-ALL/casalyin-server
git checkout -b feature/i18n-extract-hardcoded-text
```

### Step 2: 编写提取脚本
创建 `scripts/extract-i18n.js`：

**核心逻辑**：
1. 遍历 `src/` 下所有 `.tsx`, `.ts`, `.jsx`, `.js` 文件
2. 解析 AST，识别：
   - JSX 文本节点：`<div>中文</div>`
   - 字符串字面量：`const text = "中文"`
   - 模板字符串：`` `中文${var}` ``
3. 排除：
   - 注释
   - console.log
   - import/export 语句
   - 已使用 `t()` 的文案
4. 生成 key：`${fileName}.${context}.${hash}`
5. 替换原文为 `t('key')`
6. 更新 `zh-CN.json`

### Step 3: 执行提取
```bash
node scripts/extract-i18n.js --dry-run  # 预览
node scripts/extract-i18n.js --execute  # 执行
```

### Step 4: 人工审查
1. 检查 git diff
2. 优化 key 命名
3. 处理边界情况

### Step 5: 补充翻译
1. 使用 AI 翻译工具批量翻译 `en.json` 和 `es-ES.json`
2. 人工校对关键文案

### Step 6: 测试验证
1. 启动开发服务器
2. 切换语言测试
3. 检查是否有遗漏或错误

### Step 7: 提交
```bash
git add .
git commit -m "feat: extract hardcoded Chinese text to i18n files"
git push origin feature/i18n-extract-hardcoded-text
```

## Key 命名规范

### 格式
```
{module}.{component}.{context}.{type}
```

### 示例
```json
{
  "product.form.advancedSettings": "高级设置",
  "product.detail.usage.searchPlaceholder": "搜索使用方法",
  "product.editor.saveButton": "保存",
  "product.list.deleteConfirm": "确认删除？"
}
```

### 规则
1. 使用小驼峰（camelCase）
2. 层级不超过 4 层
3. 最后一层描述用途（button/label/placeholder/message/title）
4. 相同文案复用同一个 key

## 风险控制

### 避免代码冲突
1. **独立分支**：`feature/i18n-extract-hardcoded-text`
2. **不修改业务逻辑**：只做文案提取
3. **小步提交**：按模块分批提交
4. **及时同步**：定期 rebase main 分支

### 回滚方案
1. 保留原始代码备份
2. 每个模块单独 commit
3. 出问题可 revert 单个 commit

## 时间估算

- 编写脚本：2-3 小时
- 执行提取：30 分钟
- 人工审查：3-4 小时
- 补充翻译：2-3 小时
- 测试验证：1-2 小时

**总计：9-13 小时**

## 交付物

1. ✅ 独立分支：`feature/i18n-extract-hardcoded-text`
2. ✅ 提取脚本：`scripts/extract-i18n.js`
3. ✅ 更新的国际化文件：`zh-CN.json`, `en.json`, `es-ES.json`
4. ✅ 修改后的源代码（所有硬编码文案已替换）
5. ✅ 测试报告（语言切换验证）

## 后续优化

1. 集成到 CI/CD：检测新增硬编码文案
2. 添加 ESLint 规则：禁止硬编码中文
3. 建立翻译流程：新增文案自动通知翻译团队
