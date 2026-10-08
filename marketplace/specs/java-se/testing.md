# 单元测试规范

## 测试框架

使用 JUnit 5（JUnit Jupiter）。

## 规则

### 测试结构（Given-When-Then）

```java
@Test
void shouldReturnUserWhenIdExists() {
    // Given：准备数据
    UserRepository repo = new InMemoryUserRepository();
    UserService service = new UserService(repo);

    // When：执行操作
    User result = service.getUserById(1L);

    // Then：验证结果
    assertThat(result).isNotNull();
    assertThat(result.getName()).isEqualTo("Alice");
}
```

### 命名

测试方法名用自然语言描述测试场景，不使用驼峰：

```java
// 推荐
void shouldReturnNullWhenUserNotFound()
void throwExceptionWhenNameIsBlank()
void addItemToCartSuccessfully()

// 避免
void testGetUserById()
void saveTest()
```

### 断言

- 使用 AssertJ 流式断言：`assertThat(result).isNotNull().isEqualTo(expected)`
- 单个值断言用 `assertEquals(expected, actual)`
- 布尔断言用 `assertTrue()` / `assertFalse()`
- 异常断言用 `assertThrows()`

```java
// 正确
assertThrows(IllegalArgumentException.class, () -> service.getUserById(null));

// 错误
try {
    service.getUserById(null);
    fail("Should throw exception");
} catch (Exception e) { }
```

### 测试隔离

- 测试之间禁止共享可变状态
- 测试使用数据构造器（Builder）或 `@BeforeEach` 准备数据
- 测试结束后清理外部资源（文件、临时表）

### 覆盖要求

- 核心业务方法：分支覆盖率达到 80%+
- 异常分支必须有对应测试用例
- 禁止无断言的空测试

### 禁止事项

- 禁止 `Thread.sleep()` 等待异步结果，用 `await()` 或重试机制
- 禁止依赖外部网络（使用 Mock）
- 禁止测试私有方法（测试公开行为即可）
