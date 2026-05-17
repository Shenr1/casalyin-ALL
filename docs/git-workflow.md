# Git 分支管理规范

## 版本封存

**当前稳定版本：v1.0.0-stable**

- 前端（casalyin-Headless）：v1.0.0-stable (commit baf5cbd)
- 后端（casalyin-java）：v1.0.0-stable
- 后台管理（casalyin-server）：v1.0.0-stable

## 分支策略

### main 分支（生产环境）

- **用途**：生产环境自动部署分支
- **规则**：
  - 只接受来自 `develop` 分支的 merge
  - 每次 merge 到 main 后自动触发生产环境部署
  - 禁止直接在 main 上开发
  - 每次发版后打 tag（格式：`vX.Y.Z-stable`）

### develop 分支（开发环境）

- **用途**：日常开发主分支
- **规则**：
  - 所有新功能开发从 develop 创建 feature 分支
  - feature 分支完成后 merge 回 develop
  - develop 分支稳定后 merge 到 main 发版

### feature 分支（功能开发）

- **命名规范**：`feature/功能描述`
  - 例：`feature/product-search`
  - 例：`feature/user-profile`
  - 例：`feature/payment-integration`

- **工作流程**：
  ```bash
  # 1. 从 develop 创建 feature 分支
  git checkout develop
  git pull origin develop
  git checkout -b feature/功能描述
  
  # 2. 开发并提交
  git add .
  git commit -m "feat: 功能描述"
  
  # 3. 推送到远程
  git push origin feature/功能描述
  
  # 4. 功能完成后 merge 回 develop
  git checkout develop
  git pull origin develop
  git merge feature/功能描述
  git push origin develop
  
  # 5. 删除 feature 分支
  git branch -d feature/功能描述
  git push origin --delete feature/功能描述
  ```

### hotfix 分支（紧急修复）

- **命名规范**：`hotfix/问题描述`
  - 例：`hotfix/login-error`
  - 例：`hotfix/image-broken`

- **工作流程**：
  ```bash
  # 1. 从 main 创建 hotfix 分支
  git checkout main
  git pull origin main
  git checkout -b hotfix/问题描述
  
  # 2. 修复并提交
  git add .
  git commit -m "fix: 问题描述"
  
  # 3. merge 回 main 和 develop
  git checkout main
  git merge hotfix/问题描述
  git push origin main
  
  git checkout develop
  git merge hotfix/问题描述
  git push origin develop
  
  # 4. 打 tag
  git tag -a vX.Y.Z-hotfix -m "紧急修复：问题描述"
  git push origin vX.Y.Z-hotfix
  
  # 5. 删除 hotfix 分支
  git branch -d hotfix/问题描述
  git push origin --delete hotfix/问题描述
  ```

## Commit Message 规范

### 格式

```
<type>: <subject>

<body>（可选）

<footer>（可选）
```

### Type 类型

- `feat`: 新功能
- `fix`: Bug 修复
- `docs`: 文档更新
- `style`: 代码格式调整（不影响功能）
- `refactor`: 重构（不是新功能也不是 Bug 修复）
- `perf`: 性能优化
- `test`: 测试相关
- `chore`: 构建工具、依赖更新等

### 示例

```
feat: 添加产品搜索功能

- 实现前端搜索 UI
- 集成后端搜索 API
- 添加搜索结果分页

Closes #123
```

## 发版流程

### 1. 准备发版

```bash
# 确保 develop 分支所有功能已完成并测试通过
git checkout develop
git pull origin develop

# 运行 E2E 测试
cd casalyin-server
npm run test:e2e
```

### 2. Merge 到 main

```bash
git checkout main
git pull origin main
git merge develop
```

### 3. 打 tag

```bash
# 版本号规则：vX.Y.Z
# X: 主版本号（重大变更）
# Y: 次版本号（新功能）
# Z: 修订号（Bug 修复）

git tag -a v1.1.0 -m "版本 1.1.0：新增产品搜索功能"
git push origin v1.1.0
```

### 4. 推送到生产

```bash
git push origin main
# 生产环境会自动部署
```

### 5. 验证生产环境

- 访问 https://app.casalyin.com 验证前端
- 访问 https://admin.casalyin.com 验证后台管理
- 检查关键功能是否正常

### 6. 如果发现问题

- 立即创建 hotfix 分支修复
- 或者回滚到上一个稳定版本：
  ```bash
  git checkout main
  git reset --hard v1.0.0-stable
  git push origin main --force
  ```

## 当前开发计划

### 下一个版本：v1.1.0

**计划功能**：
- [ ] 产品搜索功能
- [ ] 用户个人中心
- [ ] 订单管理优化

**开发分支**：
- `feature/product-search`
- `feature/user-profile`
- `feature/order-management`

### 测试数据上传

**当前阶段**：上传真实数据到生产环境进行测试

**注意事项**：
- 在 develop 分支上进行数据上传测试
- 发现 Bug 立即在 feature 分支修复
- 修复完成后 merge 回 develop
- develop 稳定后再 merge 到 main

## 团队协作

### Claude CLI Team（本地开发）

- 在 feature 分支上开发
- 完成后 commit 但不 push
- 通知 Hermes 进行 code review 和 push

### Hermes（运维 + 发版）

- Code review feature 分支
- Push 到远程仓库
- Merge 到 develop
- 发版时 merge develop 到 main
- 生产环境部署和监控

## 回滚策略

### 快速回滚

```bash
# 1. 查看历史 tag
git tag -l

# 2. 回滚到指定版本
git checkout main
git reset --hard v1.0.0-stable
git push origin main --force

# 3. 通知团队
# 在 Slack/Discord 通知团队回滚操作
```

### 数据库回滚

```bash
# 如果涉及数据库变更，需要手动回滚 Flyway 迁移
# 1. SSH 到服务器
ssh -i ~/Downloads/casalyin_home.pem root@47.77.200.69

# 2. 连接数据库
mysql -h rm-rj9575p29a07omi0q.mysql.rds.aliyuncs.com -u casalyin -p casalyin_admin_v1

# 3. 查看 Flyway 历史
SELECT * FROM flyway_schema_history ORDER BY installed_rank DESC LIMIT 5;

# 4. 手动回滚（删除对应版本的记录，并执行反向 SQL）
# 注意：这是高危操作，必须提前备份数据库
```

## 注意事项

1. **永远不要直接在 main 分支开发**
2. **每次 push 到 main 都会触发生产部署**
3. **发版前必须运行 E2E 测试**
4. **重要变更必须先在 develop 分支验证**
5. **生产环境问题优先使用 hotfix 分支修复**
6. **每次发版后必须打 tag**
7. **回滚操作必须通知团队**

## 相关文档

- [CLAUDE.md](../CLAUDE.md) - 项目规范和团队协作
- [E2E 测试规范](testing/test-strategy.md)
- [部署文档](deployment/README.md)
