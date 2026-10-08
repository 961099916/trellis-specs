# 命名规范

## 数据库命名

| 对象 | 规则 | 示例 |
|------|------|------|
| 表名 | 小写下划线，单数名词 | `user_account`、`order_detail` |
| 字段名 | 小写下划线 | `user_id`、`created_at` |
| 索引名 | `idx_` + 表名 + 字段 | `idx_user_account_username` |
| 唯一索引 | `uk_` + 表名 + 字段 | `uk_user_account_phone` |

## Java 命名

| 对象 | 规则 | 示例 |
|------|------|------|
| 类名 | 大驼峰，名词 | `UserController`、`OrderService` |
| 方法名 | 小驼峰，动词或动词短语 | `getUserById`、`saveOrder` |
| 变量名 | 小驼峰，见名知意 | `userId`、`orderList` |
| 常量 | 全大写下划线 | `MAX_RETRY_COUNT`、`DEFAULT_PAGE_SIZE` |
| 包名 | 全小写，单数名词 | `com.example.project` |

## MyBatis 命名

| 对象 | 规则 | 示例 |
|------|------|------|
| Mapper 接口 | 大驼峰 + Mapper | `UserMapper`、`OrderMapper` |
| Mapper XML | 与接口同名 | `UserMapper.xml` |
| 方法名（XML） | 小驼峰 | `selectById`、`insertBatch` |

## 方法命名约定

### 查询类
- `get` — 按主键/唯一键查询，返回单个对象，查不到返回 null
- `find` — 按条件查询，返回 List 或 Optional
- `list` — 列表查询，返回 List
- `count` — 计数，返回 long 或 int
- `exists` — 判断是否存在，返回 boolean

### 增删改类
- `save` — 新增单个
- `saveBatch` — 批量新增
- `update` — 按主键更新
- `delete` — 按主键删除
- `remove` — 软删除（更新 deleted 字段）

## 规则

1. 类名、方法名、变量名必须见名知意，禁止用 `a`、`b`、`temp`、`data` 等无意义命名
2. 布尔变量禁用 `is` 前缀（与数据库字段映射时注意，如 `isDeleted` → `deleted`）
3. 集合变量用复数名词或 `List`/`Set`/`Map` 后缀：`users`、`userList`
4. DTO/VO/BO 统一加后缀：`UserDTO`、`UserVO`、`UserQueryBO`
