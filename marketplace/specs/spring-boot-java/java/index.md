# Java 编码规范

> 来源：~/.htcode/skills/code-standards/_languages/java.md

## 分层职责

| 层 | 职责 | 禁止 |
|---|---|---|
| Controller | 参数校验、组装响应、路由 | 写业务逻辑、直接调 Mapper |
| Service | 业务逻辑、事务边界、编排 | 出现 HttpServletRequest、拼 SQL |
| Manager（可选） | 外部系统适配、缓存编排 | 含业务判断 |
| Mapper | 单表/关联查询，**只做数据存取** | 写业务判断、多表业务拼装 |

判断标准：Controller 方法超过 20 行、Service 方法超过 80 行 → 分层大概率错了。

## 命名

| 类型 | 规则 | 示例 |
|---|---|---|
| 类名 | 大驼峰，名词 | `OrderServiceImpl` |
| 方法名 | 小驼峰，动词开头 | `cancelExpiredOrders()` |
| 变量名 | 小驼峰，表意完整 | `pendingOrders`（非 `list`） |
| 常量 | 全大写下划线 | `MAX_RETRY_COUNT` |
| 枚举 | 类名 `XxxEnum`，值全大写 | `OrderStatusEnum.PAID` |
| 布尔 | `is/has/can` 前缀 | `isValid`、`hasPermission` |
| DTO | `XxxReq` / `XxxResp` / `XxxDTO` | `OrderCreateReq` |

**禁止**：拼音缩写（`ddxx`）、单字母（循环变量除外）、`data`/`info`/`temp` 这类无信息量后缀。

## 复杂度控制

```
方法 > 80 行        → 拆分
if/else 嵌套 > 3 层 → 卫语句 / 策略模式 / 提前 return
参数 > 5 个          → 参数对象
类 > 500 行          → 拆分职责
圈复杂度 > 15        → 必须重构
```

卫语句示范：
```java
// ❌ 嵌套
if (order != null) {
    if (order.isPaid()) {
        if (!order.isRefunded()) { /* 业务逻辑 */ }
    }
}
// ✅ 卫语句
if (order == null) throw new BusinessException("订单不存在");
if (!order.isPaid()) throw new BusinessException("订单未支付");
if (order.isRefunded()) throw new BusinessException("订单已退款");
// 业务逻辑
```

## 空值与判等

- **包装类用 `Objects.equals()`**，禁止 `==`（Integer 缓存 -128~127 会出隐蔽 Bug）。
- **集合用 `CollectionUtils.isEmpty()`**，返回空集合而非 `null`。
- **Optional 只用于返回值**，禁止用作入参或字段。
- 字符串判空用 `StringUtils.hasText()`（同时排除 `""` 与纯空格）。

## 异常

```java
// ✅ 自定义业务异常，带错误码
throw new BusinessException(OrderErrorCode.STOCK_NOT_ENOUGH, "商品[%s]库存不足", productId);

// ❌ 禁止
throw new RuntimeException("error");          // 无错误码，前端无法处理
catch (Exception e) { }                        // 吞异常
catch (Exception e) { e.printStackTrace(); }   // 替代日志
```

原则：
- 业务分支用**业务异常**（可预期），技术故障用**系统异常**（不可预期）。
- 异常信息中含**关键上下文**（订单号、用户 ID），便于排查。
- 在**服务边界**（ControllerAdvice）统一捕获转换，不要层层 try-catch。

## 日志

```java
// ✅ 占位符 + 上下文
log.info("订单创建成功, orderNo={}, userId={}", orderNo, userId);
log.error("支付回调处理失败, orderNo={}", orderNo, e);   // 异常放最后参数

// ❌
log.info("订单创建成功" + orderNo);        // 字符串拼接
log.info("订单创建成功, orderNo=" + orderNo);
log.error(e.getMessage());                 // 丢堆栈
log.info("用户密码:{}", password);         // 敏感信息
```

| 级别 | 用途 |
|---|---|
| ERROR | 影响流程、需要人工介入 |
| WARN | 可自愈的异常分支（重试成功、降级生效） |
| INFO | 关键业务节点（创建/支付/取消）、外部调用耗时 |
| DEBUG | 排查用明细，生产默认关闭 |

**禁止**：日志打印密码/身份证/银行卡/token；高频循环内打日志。

## 并发与事务

- 共享可变状态必须明确保护（`ConcurrentHashMap` / `AtomicX` / 锁 / ThreadLocal 隔离）。
- **禁止用静态变量存业务状态**（多实例下不可靠）。
- 先查后写必有并发漏洞 → 用条件原子更新：
  ```sql
  UPDATE product SET stock = stock - #{qty} WHERE id = #{id} AND stock >= #{qty}
  ```
  判断影响行数，为 0 即库存不足。
- **事务内不做远程调用**（HTTP、MQ、Redis 写入尽量外移）——长事务占连接。
- 事务方法用 `@Transactional(rollbackFor = Exception.class)`（默认只回滚 RuntimeException）。

## 集合与流

- 明确初始容量：已知大小用 `new ArrayList<>(n)`，避免多次扩容。
- `Map` 遍历用 `entrySet()`，不要 `keySet()` 再 `get()`。
- Stream 超过 3 个中间操作或含复杂 lambda → 改写为普通循环，可读性优先。

## 注释

**只写 why，不写 what。**

```java
// ❌ 废话
i++;   // i 加 1

// ✅ 解释意图
retryCount++;  // 网络抖动重试，最多 3 次，超过标记失败
```

必注释场景：非直觉的算法、绕过已知缺陷的 workaround、兼容历史数据的特殊分支、并发/事务的隐含约束。
禁止：注释掉的代码块、TODO 无责任人无日期、Javadoc 只重复方法名。

## 日期与金额

- 金额一律 `BigDecimal`，比较用 `compareTo()`，禁止 `equals()`（`1.0` 与 `1.00` 不等）。
- 日期用 `LocalDateTime`/`LocalDate`，禁止 `Date`+`SimpleDateFormat`（非线程安全）。
- 时间戳字段统一 UTC 存储，展示层转本地时区。
