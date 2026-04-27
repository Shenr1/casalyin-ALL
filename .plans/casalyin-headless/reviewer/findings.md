# reviewer - 发现索引

> 纯索引——每个条目应简短（Status + Report 链接 + Summary）。

---

## review-T9-store-detail
- Status: complete
- Verdict: [WARN]（H-1 由 team-lead 决策豁免，其余已修复）
- Summary: C-1/H-2/H-3 已修复；H-1（全客户端渲染）team-lead 确认为全站现状，T9 不单独处理；残留 MEDIUM：sanitize.ts SSR 返回空字符串

## review-sidebar-unification
- Status: complete
- Verdict: [OK]
- Summary: 无 CRITICAL/HIGH；4 个 MEDIUM（Descriptions key 重复风险、ProductEditor draft/create 未迁移 CDL、draft 模式缺 createdAt/updatedAt）；3 个 LOW（空回调残留、颜色映射不一致、meta 构建风格差异）
