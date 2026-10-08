# Spring Boot 规范

> 来源：~/.htcode/skills/code-standards/_languages/springboot.md

## Controller

```java
@RestController
@RequestMapping("/api/order")
@RequiredArgsConstructor
public class OrderController {

    private final OrderService orderService;

    @PostMapping
    public Result<OrderCreateResp> create(@Valid @RequestBody OrderCreateReq req) {
        return Result.success(orderService.createOrder(req));
    }
}
```

规则：
- **统一返回 `Result<T>`**，不要直接返回裸对象或 `ResponseEntity`。
- 入参校验用 `@Valid` + Bean Validation（`@NotNull` `@NotBlank` `@Size` `@Pattern`），**不要手写 if 判空**。
- Controller **不含业务逻辑**，一行调用 Service 是常态。
- URL 用小写中划线（`/api/vip-provider/contract`），**必须带版本前缀**，改签名时升版本。
- 只暴露必要字段：入参用 `XxxReq`，出参用 `XxxResp`，**禁止直接把 Entity 当出入参**（会泄露内部字段）。

## 参数校验

```java
@Data
public class OrderCreateReq {
    @NotNull(message = "用户ID不能为空")
    private Long userId;

    @NotEmpty(message = "商品列表不能为空")
    @Size(max = 50, message = "单次最多50个商品")
    private List<OrderItemReq> items;

    @Pattern(regexp = "^1[3-9]\\d{9}$", message = "手机号格式不正确")
    private String mobile;
}
```

- 校验失败统一由 `@RestControllerAdvice` 转 `Result.fail(400, msg)`。
- 金额类字段加 `@Digits(integer = 10, fraction = 2)`。

## 全局异常处理

```java
@RestControllerAdvice
@Slf4j
public class GlobalExceptionHandler {

    @ExceptionHandler(BusinessException.class)
    public Result<Void> handleBusiness(BusinessException e) {
        log.warn("业务异常, code={}, msg={}", e.getCode(), e.getMessage());
        return Result.fail(e.getCode(), e.getMessage());
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public Result<Void> handleValid(MethodArgumentNotValidException e) {
        String msg = e.getBindingResult().getFieldError().getDefaultMessage();
        return Result.fail(400, msg);
    }

    @ExceptionHandler(Exception.class)
    public Result<Void> handleUnknown(Exception e) {
        log.error("系统异常", e);           // 全量堆栈
        return Result.fail(500, "系统繁忙，请稍后重试");  // 不暴露内部信息
    }
}
```

**安全约束**：系统异常对外只回通用文案，禁止把 `e.getMessage()` 直接抛给前端。

## 事务

```java
@Transactional(rollbackFor = Exception.class, propagation = Propagation.REQUIRED)
public void createOrder(OrderCreateReq req) { ... }
```

- 必须写 `rollbackFor = Exception.class`（默认只回滚 RuntimeException）。
- **事务方法要 public**，同类内部自调用会绕过代理导致事务失效（用 self-injection 或拆到另一个类）。
- **事务内禁止远程调用**（HTTP/MQ/长耗时 IO），会长时间占数据库连接。
- 只读查询加 `@Transactional(readOnly = true)`。
- 大批量处理拆成小事务，避免长事务与锁等待。

## 配置

- 配置按环境分 `application-{env}.yml`，**敏感配置（密码、密钥）走配置中心或环境变量**，禁止入库到代码。
- 自定义配置用 `@ConfigurationProperties` + 前缀，**不要用散落的 `@Value`**。
- 配置项必须有默认值或启动时校验，缺配置要 fail-fast。

## 依赖注入

- 一律**构造器注入**（配合 `@RequiredArgsConstructor` + `final` 字段），禁止 `@Autowired` 字段注入（无法 final、难测试、易循环依赖）。
- 循环依赖是设计问题，**拆服务**，不要用 `@Lazy` 掩盖。

## 接口幂等

写操作（创建订单、支付、扣减）必须支持幂等：

| 方案 | 适用 |
|---|---|
| 唯一索引 + 业务单号 | 创建类，最可靠 |
| Token/幂等号 + Redis 预占 | 前端重复提交 |
| 状态机校验（已支付则拒绝） | 状态流转类 |
| 乐观锁 version | 更新类 |

## 日志与链路

- 每个请求带 `traceId`（MDC），日志中自动输出。
- 外部调用记录：目标、耗时、返回码；**慢调用（>1s）打 WARN**。
- 定时任务必须记录开始/结束/影响行数。

## 常见陷阱

| 陷阱 | 表现 | 规避 |
|---|---|---|
| `@Transactional` 自调用失效 | 异常不回滚 | 拆类或注入自身代理 |
| `@Async` 与 `@Transactional` 同方法 | 事务不生效 | 拆分 |
| MyBatis 一级缓存 | 同一 SqlSession 读到旧值 | 禁用或明确 flush |
| 拦截器放行路径写错 | 401/404 | 明确配置 exclude，加测试 |
| `LocalDateTime` JSON 序列化 | 格式不一致 | 全局配置 `spring.jackson` |
| 大对象放 Session / ThreadLocal 未清理 | 内存泄漏 | finally 中 remove |
