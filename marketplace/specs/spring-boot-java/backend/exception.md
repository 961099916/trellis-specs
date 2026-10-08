# 异常处理规范

## 统一异常体系

本项目使用三级异常体系：

```
RuntimeException
├── BusinessException     # 业务异常（用户可感知，需提示）
├── SystemException       # 系统异常（代码/基础设施问题，不直接提示用户）
└── ValidationException   # 参数校验异常（继承自 BusinessException）
```

## 规则

### 抛出业务异常

业务规则不满足时抛出 `BusinessException`，禁止直接返回错误码或 null。

```java
// 正确：抛业务异常
if (user == null) {
    throw new BusinessException("用户不存在");
}

// 错误：返回 null 让调用方判断
return null;
```

### Controller 层禁止捕获业务异常后返回错误信息

统一异常处理器负责此事：

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(BusinessException.class)
    public Result<Void> handleBusinessException(BusinessException e) {
        log.warn("业务异常: {}", e.getMessage());
        return Result.fail(e.getCode(), e.getMessage());
    }

    @ExceptionHandler(Exception.class)
    public Result<Void> handleException(Exception e) {
        log.error("系统异常", e);
        return Result.fail("系统异常，请稍后重试");
    }
}
```

### 日志规范

- `log.warn` — 业务异常、预期内的边界情况
- `log.error` — 系统异常、未预期的错误，**必须带异常堆栈**
- 禁止 `e.printStackTrace()`、禁止 `log.info` 打印异常堆栈

### 禁止吞掉异常

```java
// 错误：吞异常
try {
    doSomething();
} catch (Exception e) {
    // 什么都不做
}

// 正确：至少记录
try {
    doSomething();
} catch (Exception e) {
    log.error("操作失败", e);
    throw new RuntimeException("操作失败", e);
}
```

## 统一响应格式

所有 Controller 返回统一 JSON 格式：

```json
{
  "code": 200,
  "message": "success",
  "data": { ... }
}
```

错误响应示例：

```json
{
  "code": 40001,
  "message": "用户不存在",
  "data": null
}
```

**参考文件：** `src/main/java/{package}/common/Result.java`、`GlobalExceptionHandler.java`
