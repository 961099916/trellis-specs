# 日志规范

## 日志框架

使用 Slf4j API，禁止直接使用 Log4j / Logback 具体实现类。

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

// 类静态成员
private static final Logger log = LoggerFactory.getLogger(MyService.class);
```

## 日志级别使用场景

| 级别 | 场景 |
|------|------|
| `ERROR` | 系统级错误，需人工介入；异常堆栈必须打印 |
| `WARN` | 业务异常、边界条件、外部依赖超时/降级 |
| `INFO` | 系统启动/停止、关键业务节点（订单创建、支付完成） |
| `DEBUG` | 开发调试信息，生产环境默认关闭 |
| `TRACE` | 更细粒度的调试信息，生产环境关闭 |

## 规则

1. **禁止 `System.out.println()`、`System.err.println()`**
2. **禁止 `e.printStackTrace()`**，用 `log.error("...", e)` 代替
3. 日志消息用占位符而非字符串拼接：`log.info("user={}, status={}", userId, status)`
4. 日志内容禁止包含密码、Token、手机号、身份证号等敏感信息
5. 生产环境 `INFO` 级别，生产调试用 `DEBUG`，压测用 `WARN`

## 敏感信息脱敏

```java
// 手机号脱敏：138****5678
String maskPhone(String phone) { ... }

// 身份证脱敏：110***********1234
String maskIdCard(String idCard) { ... }
```

## 反模式

```java
// 错误：字符串拼接
log.info("用户 " + userId + " 登录成功");

// 正确：占位符
log.info("用户 {} 登录成功", userId);
```
