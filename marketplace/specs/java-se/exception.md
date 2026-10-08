# 异常处理规范

## 异常分类

| 类型 | 使用场景 | 示例 |
|------|----------|------|
| `RuntimeException` | 程序错误，可不声明 | `NullPointerException`、`IllegalArgumentException` |
| `Exception`（受检） | 调用方必须处理的异常 | `IOException`、`SQLException` |
| 自定义业务异常 | 业务规则不满足时 | `BusinessException`、`ValidationException` |

## 规则

### 资源关闭

所有实现了 `AutoCloseable` 的资源必须使用 try-with-resources：

```java
// 正确
try (BufferedReader reader = new BufferedReader(new FileReader(path));
     BufferedWriter writer = new BufferedWriter(new FileWriter(out))) {
    // ...
}

// 错误：手动关闭，容易遗漏
BufferedReader reader = null;
try {
    reader = new BufferedReader(new FileReader(path));
} finally {
    if (reader != null) reader.close();
}
```

### 异常链

重新抛出异常时保留原异常上下文：

```java
// 正确
throw new ServiceException("业务处理失败", e);

// 错误：吞掉原异常
throw new ServiceException("业务处理失败");
```

### 禁止事项

- 禁止空 catch 块（至少记录日志）
- 禁止 `catch (Exception e)` 捕获所有异常后无差别处理
- 禁止 `e.printStackTrace()`
- 禁止在异常处理中抛出新异常而不记录原异常

## 自定义异常示例

```java
public class BusinessException extends RuntimeException {
    private final String code;

    public BusinessException(String code, String message) {
        super(message);
        this.code = code;
    }

    public BusinessException(String code, String message, Throwable cause) {
        super(message, cause);
        this.code = code;
    }

    public String getCode() { return code; }
}
```
