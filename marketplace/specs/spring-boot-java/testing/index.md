# 单元测试规范

> 来源：~/.htcode/skills/code-standards/_languages/testing.md

## 测试什么

| 必测 | 可不测 |
|---|---|
| 核心业务逻辑（计算、状态流转、校验规则） | 纯 getter/setter |
| 分支与边界（null、空集合、极值） | 框架自动生成代码 |
| 异常路径（预期抛什么异常） | 配置类 |
| 金额/权限/幂等 相关逻辑 | 第三方 SDK 本身 |

目标不是覆盖率数字，而是**需求里的每个判断条件都有对应断言**。

## 命名：应该描述"场景+预期"

```java
// ❌
@Test void test1() {}
@Test void testCreateOrder() {}

// ✅ 方法_场景_预期
@Test
@DisplayName("创建订单-库存不足-抛业务异常且不落库")
void createOrder_insufficientStock_throwAndNotInsert() {}
```

## 结构：Given-When-Then

```java
@Test
@DisplayName("取消订单-已支付-允许取消并释放库存")
void cancelOrder_paid_releaseStock() {
    // Given：准备数据与 mock
    Order order = OrderFixture.paidOrder("SO001", 1L, 2);
    when(orderMapper.selectByOrderNo("SO001")).thenReturn(order);
    when(inventoryService.release(1L, 2)).thenReturn(true);

    // When：执行
    orderService.cancel("SO001");

    // Then：断言行为
    assertThat(order.getStatus()).isEqualTo(OrderStatusEnum.CANCELED);
    verify(inventoryService).release(1L, 2);
    verify(orderMapper).updateById(order);
}
```

## 断言（硬性要求）

- **每个测试必须有断言**。只调用不断言的测试是无效测试：
  ```java
  // ❌ 无效
  @Test void testCreateOrder() { orderService.createOrder(req); }

  // ✅ 有效
  @Test void createOrder_success() {
      OrderResp resp = orderService.createOrder(req);
      assertThat(resp.getOrderNo()).isNotBlank();
      verify(orderMapper).insert(any());
  }
  ```
- 用 AssertJ（`assertThat`），链式可读性强于 JUnit 原生。
- **断言行为而非实现**：优先验证结果状态与关键调用（`verify`），不要 mock 一切后只验证"方法被调用了"。
- 异常断言用 `assertThatThrownBy(...).isInstanceOf(XxxException.class).hasMessageContaining("...")`。

## Mock 的边界

**可以 mock**：外部服务、数据库访问、消息队列、时间、随机数。
**不要 mock**：被测类自身、纯函数、值对象。

反模式：把整个 Service 的每一步都 mock 掉，测试变成了"验证调用顺序"，实现一改测试就崩，且不验证任何真实行为——这种测试没有价值。

## 独立性

- 测试之间**不能有顺序依赖**，不能依赖上一个测试留下的数据库状态。
- 每个测试自己准备数据（Fixture / Builder）。
- 涉及数据库的用 `@Transactional` 回滚或测试容器内建数据，**禁止连开发/测试环境共享库**。
- 不依赖 `Thread.sleep()` 等待异步——用 `Awaitility` 或显式同步点。

## 边界清单（必须覆盖）

```
[ ] null 入参
[ ] 空字符串 / 空集合
[ ] 0 / 负数 / 极大值
[ ] 金额精度（1.00 vs 1.0）
[ ] 并发重复调用（幂等）
[ ] 时间边界（跨天、月末、闰年）
[ ] 权限不足场景
```

## 数据准备

用 Builder/Fixture 造数据，**不要在测试里手写几十行 setter**：

```java
Order order = OrderFixture.builder()
        .orderNo("SO001")
        .userId(1001L)
        .status(OrderStatusEnum.PAID)
        .amount(new BigDecimal("99.90"))
        .build();
```
