# Hermes ↔ Claude CLI 任务交接目录

这个目录用于 Hermes 和 Claude CLI Team 之间的任务交接文档存放。

## 工作流程

1. **Hermes 发现问题**
   - 自动判断问题类型
   - 小问题（配置、环境、容器）直接修复
   - 复杂问题（代码、架构）创建任务文件

2. **用户通知 Claude CLI Team**
   - 用户在 Claude CLI Team 中说："有新任务，请读取 .hermes-tasks/xxx.md"
   - team-lead 读取任务文件
   - 根据任务内容和代码情况自己决策复杂度
   - 简单任务直接改完标记 DONE
   - 复杂任务走完整流程（reviewer + e2e-tester）后标记 DONE
   - **注意**：本地 commit 即可，不要 push

3. **Hermes 验证和提交**
   - 检测到任务状态为 DONE
   - git pull 拉取 Claude CLI Team 的 commit
   - 验证代码变更合理性
   - git push origin main
   - 等待 CI/CD 自动部署（约 3-5 分钟）
   - 验证部署结果（容器状态、API 接口、数据库连接）
   - 验证通过 → 归档任务到 archive/
   - 验证失败 → 更新任务状态为 PENDING，添加失败信息

## 文件命名规则

格式：`YYYYMMDD-HHMM-类型-描述.md`

示例：
- `20260430-1400-bug-redis-serialization.md`
- `20260430-1530-feature-email-template.md`
- `20260430-1600-config-nginx-proxy.md`

## 任务状态

- `PENDING`：待处理
- `IN_PROGRESS`：处理中（可选）
- `DONE`：已完成，等待 Hermes 验证
- `VERIFIED`：验证通过，已归档

## 任务模板

参考 `TEMPLATE.md`

## 归档规则

验证通过的任务会自动归档到 `archive/YYYY-MM/` 目录。
