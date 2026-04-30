# 任务：将邮件服务从 QQ SMTP 迁移到 Resend HTTP API

**创建时间**：2026-04-29 12:06  
**优先级**：高  
**状态**：TODO

## 问题描述

生产环境的邮件验证码发送功能一直失败，错误信息：
```
Authentication failed
org.springframework.mail.MailAuthenticationException: Authentication failed
Caused by: jakarta.mail.AuthenticationFailedException: 535 Error: authentication failed
```

用户尝试登录后台时，点击"发送验证码"按钮，后端返回：
```json
{
    "code": 30001,
    "level": "user",
    "msg": "发送验证码失败，请稍后重试（Authentication failed）",
    "ok": false,
    "data": null,
    "dataType": 1
}
```

## 根因分析

1. **当前配置使用 QQ 邮箱 SMTP**：
   - 配置文件：`sa-base/src/main/resources/prod/sa-base.yaml`
   - SMTP 服务器：`smtp.qq.com:465`
   - 用户名：`1024lab@qq.com`
   - 密码：`LAB1024LAB`（占位符，不是真实授权码）

2. **QQ 邮箱 SMTP 的问题**：
   - 需要 QQ 邮箱授权码（不是 QQ 密码）
   - 需要在 QQ 邮箱设置中开启 SMTP 服务
   - 面向国外用户时，QQ 邮箱可能被标记为垃圾邮件
   - 配置复杂，不适合生产环境

3. **用户需求**：
   - 项目面向国外用户
   - 需要可靠的邮件服务
   - 已有 Resend API Key：`re_U1RQpSps_2MbHMNGC2vYhwQoQrbgmmoqw`

## 临时修复（已完成）

已将 `RESEND_API_KEY` 添加到生产环境的 `.env.prod` 文件，但后端代码仍使用 Spring Mail 的 SMTP 配置，无法直接使用 Resend。

## 需要的代码修改

### 1. 添加 Resend Java SDK 依赖

在 `casalyin-java/pom.xml` 或 `sa-base/pom.xml` 中添加：
```xml
<dependency>
    <groupId>com.resend</groupId>
    <artifactId>resend-java</artifactId>
    <version>3.0.0</version>
</dependency>
```

### 2. 创建 Resend 邮件服务实现

在 `sa-base/src/main/java/net/lab1024/sa/base/module/support/mail/` 目录下创建：

**ResendMailService.java**：
```java
package net.lab1024.sa.base.module.support.mail;

import com.resend.*;
import com.resend.core.exception.ResendException;
import com.resend.services.emails.model.*;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.boot.autoconfigure.condition.ConditionalOnProperty;
import org.springframework.stereotype.Service;

@Slf4j
@Service
@ConditionalOnProperty(name = "mail.provider", havingValue = "resend")
public class ResendMailService {

    @Value("${RESEND_API_KEY:}")
    private String apiKey;

    @Value("${mail.from:noreply@yourdomain.com}")
    private String fromEmail;

    public void sendMail(String to, String subject, String htmlContent) throws ResendException {
        Resend resend = new Resend(apiKey);

        SendEmailRequest sendEmailRequest = SendEmailRequest.builder()
            .from(fromEmail)
            .to(to)
            .subject(subject)
            .html(htmlContent)
            .build();

        CreateEmailResponse response = resend.emails.send(sendEmailRequest);
        log.info("Resend email sent successfully: id={}, to={}", response.getId(), to);
    }
}
```

### 3. 修改现有的 MailService

在 `sa-base/src/main/java/net/lab1024/sa/base/module/support/mail/MailService.java` 中：

- 注入 `ResendMailService`（可选）
- 添加配置开关，根据 `mail.provider` 选择使用 SMTP 还是 Resend
- 或者直接替换 `sendMail` 方法的实现

### 4. 更新配置文件

在 `sa-base/src/main/resources/prod/sa-base.yaml` 中添加：
```yaml
mail:
  provider: resend
  from: noreply@casalyin.com  # 替换为你的域名
```

移除或注释掉原有的 `spring.mail` 配置。

### 5. 更新 .env.prod

确保生产环境的 `.env.prod` 包含：
```
RESEND_API_KEY=re_U1RQpSps_2MbHMNGC2vYhwQoQrbgmmoqw
```

（Hermes 已完成此步骤）

## 验证方式

### 本地测试
1. 在本地 `.env` 中添加 `RESEND_API_KEY`
2. 修改 `application.yaml` 设置 `mail.provider: resend`
3. 启动后端服务
4. 调用发送验证码接口：
   ```bash
   curl -X POST http://localhost:1025/api/auth/send-verify-email \
     -H 'Content-Type: application/json' \
     -d '{"email":"your-email@example.com"}'
   ```
5. 检查邮箱是否收到验证码

### 生产环境验证
1. SSH 到服务器：`ssh -i ~/Downloads/casalyin_home.pem root@47.77.200.69`
2. 拉取最新代码：
   ```bash
   cd /opt/casalyin/casalyin-java
   git pull origin main
   ```
3. 重新构建并部署：
   ```bash
   cd /opt/casalyin/casalyin-Headless/deploy
   docker-compose build backend
   docker-compose up -d backend
   ```
4. 等待服务启动（约 10 秒）
5. 测试发送验证码：
   ```bash
   curl -X POST http://localhost:1025/api/auth/send-verify-email \
     -H 'Content-Type: application/json' \
     -d '{"email":"test@example.com"}'
   ```
6. 检查响应是否为 `{"ok": true}`
7. 检查后端日志：
   ```bash
   docker logs casalyin-backend-1 --tail 50 | grep -i resend
   ```

## 技术细节

### Resend vs SMTP
- **Resend**：HTTP API，简单可靠，专为开发者设计，送达率高
- **SMTP**：传统协议，配置复杂，需要授权码，容易被标记为垃圾邮件

### Resend 发件地址要求
- 免费版可以使用 `onboarding@resend.dev`（仅用于测试）
- 生产环境需要验证自己的域名（在 Resend Dashboard 添加 DNS 记录）
- 如果暂时没有域名，可以先用测试地址

### 备选方案
如果 Resend Java SDK 有问题，可以直接用 HTTP 请求：
```java
RestTemplate restTemplate = new RestTemplate();
HttpHeaders headers = new HttpHeaders();
headers.set("Authorization", "Bearer " + apiKey);
headers.setContentType(MediaType.APPLICATION_JSON);

Map<String, Object> body = Map.of(
    "from", fromEmail,
    "to", to,
    "subject", subject,
    "html", htmlContent
);

HttpEntity<Map<String, Object>> request = new HttpEntity<>(body, headers);
ResponseEntity<String> response = restTemplate.postForEntity(
    "https://api.resend.com/emails",
    request,
    String.class
);
```

## 执行记录
- 2026-04-29 12:06 Hermes 创建任务
- 2026-04-29 12:06 Hermes 已将 RESEND_API_KEY 添加到生产环境

## 参考资料
- Resend 官方文档：https://resend.com/docs/send-with-java
- Resend Java SDK：https://github.com/resend/resend-java
- Spring Boot 邮件配置：https://docs.spring.io/spring-boot/docs/current/reference/html/messaging.html#messaging.email
