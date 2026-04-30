# 任务：验证码邮件添加 HTML 模板和 Logo

**创建时间：** 2026-04-30 08:49  
**状态：** TODO  
**类型：** Feature Enhancement  
**优先级：** P2

---

## 问题描述

当前验证码邮件是纯文本格式，需要改为 HTML 格式并添加品牌 Logo。

**当前实现：**
- 文件：`casalyin-java/casalyin-admin/src/main/java/com/casalyin/admin/module/system/login/service/EmailVerifyService.java`
- 第 52-58 行：使用纯文本发送验证码
```java
mailService.sendMail(
    "注册验证码",
    "您的验证码是：" + code + "，10分钟内有效。如非本人操作请忽略。",
    null,
    Collections.singletonList(email),
    false  // ← 当前是纯文本
);
```

---

## 需求

1. 将验证码邮件改为 HTML 格式
2. 添加 Casalyin 品牌 Logo（宽度设置为 25%）
3. 保持邮件内容简洁专业
4. 确保在各邮件客户端中显示正常

---

## 技术方案

### 方案 A：内联 HTML 模板（推荐）

直接在 `EmailVerifyService.java` 中构建 HTML 字符串：

```java
String htmlContent = String.format("""
    <!DOCTYPE html>
    <html>
    <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
    </head>
    <body style="margin: 0; padding: 20px; font-family: Arial, sans-serif; background-color: #f5f5f5;">
        <div style="max-width: 600px; margin: 0 auto; background-color: #ffffff; padding: 40px; border-radius: 8px;">
            <img src="https://admin.casalyin.com/logo.png" alt="Casalyin" style="width: 25%%; max-width: 150px; margin-bottom: 30px;">
            <h2 style="color: #333333; margin-bottom: 20px;">注册验证码</h2>
            <p style="color: #666666; font-size: 16px; line-height: 1.6;">您的验证码是：</p>
            <div style="background-color: #f0f0f0; padding: 15px; border-radius: 4px; text-align: center; margin: 20px 0;">
                <span style="font-size: 32px; font-weight: bold; color: #1890ff; letter-spacing: 5px;">%s</span>
            </div>
            <p style="color: #999999; font-size: 14px; margin-top: 20px;">验证码10分钟内有效，如非本人操作请忽略。</p>
        </div>
    </body>
    </html>
    """, code);

mailService.sendMail(
    "注册验证码",
    htmlContent,
    null,
    Collections.singletonList(email),
    true  // ← 改为 HTML 格式
);
```

**优点：**
- 无需额外配置
- 修改简单，一次性完成
- 不依赖数据库或模板引擎

### 方案 B：使用邮件模板系统

如果项目已有 `MailTemplateDao` 和模板表，可以创建数据库模板记录。

**缺点：**
- 当前数据库中没有 `t_mail_template` 表
- 需要额外的 Flyway 迁移和初始化数据
- 对于单一验证码邮件来说过于复杂

---

## 实施步骤

1. **修改 EmailVerifyService.java**
   - 构建 HTML 模板字符串
   - 将 `sendMail` 最后一个参数改为 `true`
   - 确保 Logo URL 正确（`https://admin.casalyin.com/logo.png`）

2. **验证 Logo 资源**
   - 确认 `casalyin-server` 项目中有 `public/logo.png`
   - 确认 Nginx 配置正确映射静态资源

3. **本地测试**
   - 重启后端服务
   - 触发验证码发送
   - 检查邮件客户端显示效果

4. **部署验证**
   - 提交代码
   - 触发 CI/CD 部署
   - 生产环境测试邮件发送

---

## 验证方式

### 本地验证
```bash
# 1. 重启后端
cd casalyin-java
npm run dev:stop && npm run dev

# 2. 前端触发注册流程
# 访问 http://localhost:5173/login
# 点击"注册"，输入邮箱，点击"发送验证码"

# 3. 检查邮箱
# 确认收到 HTML 格式邮件
# 确认 Logo 显示正常（宽度 25%）
# 确认验证码清晰可见
```

### 生产验证
```bash
# 访问 https://admin.casalyin.com/login
# 执行相同的注册流程
# 检查邮件效果
```

---

## 注意事项

1. **Logo URL 必须是绝对路径**
   - 使用 `https://admin.casalyin.com/logo.png`
   - 不要使用相对路径或 `file://`

2. **邮件客户端兼容性**
   - 使用内联样式（不要用 `<style>` 标签）
   - 避免使用 Flexbox/Grid（部分客户端不支持）
   - 使用 `table` 布局更安全（可选）

3. **Logo 尺寸**
   - 宽度：25%（响应式）
   - 最大宽度：150px（防止过大）
   - 保持宽高比

4. **测试环境标记**
   - `MailService` 已在非生产环境自动添加 "(测试)" 前缀
   - 无需额外处理

---

## 执行记录

- [ ] 修改 `EmailVerifyService.java`
- [ ] 验证 Logo 资源可访问
- [ ] 本地测试
- [ ] 提交代码
- [ ] 生产部署验证

---

## 完成标准

- [x] 验证码邮件为 HTML 格式
- [x] Logo 显示正常，宽度为 25%
- [x] 验证码清晰可见
- [x] 在主流邮件客户端（Gmail、Outlook、QQ邮箱）中显示正常
- [x] 本地和生产环境均验证通过
