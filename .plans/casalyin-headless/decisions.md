# casalyin-headless - 架构决策记录

> 记录每个决策及其理由。

---

## D1: 团队配置选择 3 人（frontend-dev + e2e-tester + reviewer）

- 日期: 2026-04-19
- 决策: 不含 backend-dev，因为后端由 casalyin-java 提供，不在 headless 范围内
- 理由: 后台 casalyin-server 和前台共用同一套后端，headless 团队只负责前台展示层
- 考虑过的替代方案: 加 researcher 调研 API — 因现有 docs/backend/api-conventions.md 已有完整规范，无需单独调研
