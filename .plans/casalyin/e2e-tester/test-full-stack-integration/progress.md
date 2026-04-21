# 测试执行进度

## 2026-04-20

### 环境检查
- 后端 8690：正常（/api/product/public/filter 返回 73 条产品）
- 前台 Next.js 3000：正常（返回 HTML）
- 后台 Vite 5173：未检查（测试不需要）

### 测试执行
- 第 1 次运行：T1 被错误跳过（测试代码 bug：disabledFlag 严格比较 false !== 0）
- 已修复测试代码，第 2 次运行：2 PASSED
- T1 执行 8 个断言中的 5 个（断言3/4/7 通过，断言5/6/8 因数据为空跳过）
- T2 通过，软警告：前台未展示 404 提示页面
- 非阻断性 console.error：Pinterest hydration mismatch（SSR/CSR href 不一致）
