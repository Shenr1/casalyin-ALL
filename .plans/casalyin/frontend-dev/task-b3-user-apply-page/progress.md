# B3-3 Progress

## 2026-04-12

- 读取后端 DTO：BrandApplyForm + BrandApplyVO 字段确认
- 读取现有 BrandPortal.tsx 了解已有实现
- 读取 API 层 brandApply.ts 确认接口封装完整
- 向 team-lead 确认 ContentDetailLayout 适用性
- 等待回复后开始编码

## 2026-04-12 Session — B3-3 开发

- [x] 读取后端 DTO (BrandApplyForm/BrandApplyVO) 确认字段
- [x] 读取 BrandApplyController 确认接口路径
- [x] 确认 API 层 brandApply.ts 已有完整封装
- [x] 创建 BrandApplyMyPage.tsx (4 状态: NoApply/Pending/Approved/Rejected + 申请弹窗)
- [x] 注册路由 /brands/apply-my (routes/index.tsx)
- [x] 添加 brandApplyMy.* i18n key (zh-CN + en + es-ES, 60+ key)
- [x] 更新路由清单文档 docs/frontend/routing.md
- [x] TypeScript 编译通过 (tsc --noEmit)
- [x] JSON locale 文件验证通过
- [ ] 等待 reviewer 审查


## 2026-04-13 Session — B3-2 修复品牌申请页面规范违规

### BrandApplyManagement.tsx 修复清单
- [x] message → 从 AppProvider 导入
- [x] isAdmin 判断 → permissionList.includes("brand:audit")
- [x] 6 个 Table 列标题硬编码 → t("brandApplyAdmin.*")
- [x] Card title/刷新按钮硬编码 → t()
- [x] maxWidth: 1000px 废弃 → 移除

### BrandApplyDetail.tsx 修复清单
- [x] message → 从 AppProvider 导入
- [x] isAdmin 判断 → permissionList.includes("brand:audit")
- [x] 40+ 处硬编码中文 → 全部走 t("brandApplyAdmin.*")
- [x] modal.confirm 内文案 → t() + {{name}} 插值
- [x] Modal JSX title/okText/cancelText → t()
- [x] fmt 函数参数名 t → v（避免遮蔽 useTranslation 的 t）
- [x] maxWidth: 1000px 废弃 → 移除

### i18n
- [x] 新增 brandApplyAdmin.* 命名空间 (50 key)
- [x] zh-CN / en / es-ES 三语言同步
- [x] JSON 验证通过
- [x] TypeScript 编译通过
- [x] 硬编码中文检测通过（两个文件不再出现在报告中）

