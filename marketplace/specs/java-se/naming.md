# 命名规范

## 类与接口

| 类型 | 规则 | 示例 |
|------|------|------|
| 普通类 | 大驼峰，名词 | `UserService`、`FileUtils` |
| 接口 | 大驼峰，可加 `I` 前缀或直接名词 | `Comparable`、`IUserDao` |
| 抽象类 | 大驼峰，`Abstract` 前缀 | `AbstractList`、`AbstractDao` |
| 异常类 | 大驼峰，`Exception` / `Error` 后缀 | `BusinessException`、`ParseError` |
| 枚举类 | 大驼峰，枚举值全大写下划线 | `Status`、`OrderStatus` |

## 方法与变量

- 方法名：小驼峰，动词或动词短语，见名知意
- 局部变量：小驼峰，禁止单字母（循环计数器除外：`i`、`j`、`k`）
- 成员变量：小驼峰，可加 `m` 前缀（团队统一即可）
- 常量：全大写下划线，如 `MAX_RETRY_COUNT`

## 集合与泛型

- 集合变量声明使用接口类型：`List<T>`、`Map<K,V>`，而非具体实现 `ArrayList`
- 泛型类型参数：单个用 `T`，键值用 `K`、`V`，列表用 `E`，数字用 `N`

```java
// 正确
List<User> users = new ArrayList<>();
Map<String, Integer> countMap = new HashMap<>();

// 错误
ArrayList<User> users = new ArrayList<>();
```

## 规则

1. 禁止使用拼音命名
2. 类名禁止使用动词
3. 包名全小写，单词用单数：`com.example.util`
4. 测试类与被测类同名，加 `Test` 后缀：`UserService` → `UserServiceTest`
