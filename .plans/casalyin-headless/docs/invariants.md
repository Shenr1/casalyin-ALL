# casalyin-headless - 系统不变量

> 不可违反的系统边界。违反其中任何一条 = CRITICAL Bug。

## SEO 边界

- INV-1: 所有列表页和详情页必须使用 Server Component 或 SSG 渲染（不能全客户端渲染），确保 SEO — 状态：人工检查
- INV-2: 所有图片必须有 alt 属性（无障碍 + SEO） — 状态：无测试

## 响应式边界

- INV-3: 所有页面在 375px（iPhone SE）宽度下不得出现横向溢出 — 状态：无测试
- INV-4: 移动端触控目标最小 44x44px — 状态：人工检查

## 接口契约

- INV-5: 前台调用的 API 字段名必须与 docs/api-contracts.md 一致，不能自己猜测字段名 — 状态：人工检查

## 性能边界

- INV-6: 首屏 LCP < 2.5s（Core Web Vitals 标准） — 状态：无测试
