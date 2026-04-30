# Bug: PestManagement 页面 Typography is not defined

## 状态
- [x] 问题诊断
- [ ] 代码修复
- [ ] 本地验证
- [ ] 提交代码

## 问题描述
生产环境访问 http://47.77.200.69:8080/tags/pest 报错：
```
Something went wrong.
Typography is not defined
```

## 根因分析
文件：`casalyin-server/src/pages/PestManagement.tsx`

**问题**：第 263 行使用了 `Typography.Title`，但顶部 import 语句（第 4 行）没有导入 `Typography`

```tsx
// 第 4 行 - 缺少 Typography
import { Card, Button, Space, Input, Select } from 'antd'

// 第 263 行 - 使用了 Typography
<Typography.Title level={4} style={{ margin: 0 }}>
  {t('pestManagement.title')}
</Typography.Title>
```

## 修改方案
在 `casalyin-server/src/pages/PestManagement.tsx` 第 4 行添加 `Typography` 导入：

```tsx
// 修改前
import { Card, Button, Space, Input, Select } from 'antd'

// 修改后
import { Card, Button, Space, Input, Select, Typography } from 'antd'
```

## 验证方式
1. 本地启动前端：`cd casalyin-server && npm run dev`
2. 访问 http://localhost:5173/tags/pest
3. 确认页面正常显示，无 JavaScript 错误
4. 确认标题 "病虫害管理" 正常渲染

## 执行记录
- 2026-04-30 14:47 - Hermes 诊断问题，创建任务文件
- 待 Claude CLI Team 修复代码
