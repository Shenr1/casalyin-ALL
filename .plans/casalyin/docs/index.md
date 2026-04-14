# casalyin - 知识库索引

> 动态导航地图。
> 智能体：需要在 docs/ 中查找信息时先 Read 此文件。

| 文档 | 位置 | 内容 |
|------|------|------|
| 后端接口规范 | docs/backend/api-conventions.md | HTTP 方法、路径命名、权限码、四维度接口矩阵 |
| 数据库规范 | docs/backend/database.md | Flyway 规范、字段命名约定、核心表结构 |
| UI 组件规范 | docs/frontend/ui-components.md | ContentDetailLayout、返回按钮、Table、Modal/message |
| 路由清单 | docs/frontend/routing.md | 所有路由完整清单，新增页面必须先查 |
| Editor 架构 | docs/frontend/editor-architecture.md | XxxEditor 三合一架构（mode: create/draft/detail） |
| i18n 规范 | docs/frontend/i18n.md | 禁止硬编码文案、t() 规范、三语言同步 |
| 草稿流程 | docs/business/workflow-draft.md | 状态机（DRAFT→PENDING→发布/拒绝）、免审直发 |
| 上下架规范 | docs/business/online-offline.md | disabledFlag、四维度接口、前端交互规范 |
| 角色权限 | docs/business/roles-permissions.md | BASE/COMPANY/ADMIN 权限边界 |
| VIP 体系 | docs/business/vip-monetization.md | VIP 规则、VipBanner 规范 |
| 测试策略 | docs/testing/test-strategy.md | Playwright 规范、测试账号 |
| 测试用例 | docs/testing/test-cases.md | 完整测试用例清单 |
| 架构决策 | docs/decisions/architecture-decisions.md | Dashboard 精简、XxxEditor、Flyway、上下架统一 |

## 如何使用

- 修改前端 UI → 读 ui-components.md + routing.md
- 修改接口 → 读 api-conventions.md
- 修改业务流程 → 读 workflow-draft.md 或 online-offline.md
- 修改数据库 → 读 database.md（必须用 Flyway）
- 写测试 → 读 test-strategy.md + test-cases.md
