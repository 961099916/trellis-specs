# 分层架构约定

## 三层结构

本项目采用标准三层架构：Controller → Service → Mapper。

```
controller/   # 控制层：接收请求、参数校验、调用 Service、组装响应
service/      # 服务层：处理业务逻辑、事务管理
mapper/       # 持久层：数据库 CRUD，仅做数据存取
```

## 规则

### Controller 层

- 只做三件事：参数校验、调用 Service、返回结果
- 禁止在 Controller 中写任何业务逻辑
- 使用 `@Valid` + BindingResult 做参数校验，禁止手动 if-else 校验
- 返回 `Result<T>` 统一封装类

**参考文件：** `src/main/java/{package}/controller/`

**反模式：** 在 Controller 中写 if-else 判断业务逻辑、直接操作数据库

### Service 层

- 所有业务逻辑必须在此层
- `@Transactional` 注解加在 Service 方法上，默认传播行为 `REQUIRED`
- 禁止在 Service 中直接写 SQL，使用 Mapper 代理对象
- 集合查询返回 `List<T>`，单个查询返回 `T` 或 `Optional<T>`

**参考文件：** `src/main/java/{package}/service/`

**反模式：** 在 Service 中写 JDBC SQL、绕过 Mapper 直接操作 SqlSession

### Mapper 层（MyBatis）

- Mapper 接口与 XML 文件同包同名（`UserMapper.java` + `UserMapper.xml`）
- Mapper 接口只定义方法签名，实现由 MyBatis 运行时生成
- 禁止在 XML 中写多表 join 的超长 SQL，单个 SQL不超过 50 行
- 使用 `resultMap` 而非 `resultType` 做结果映射，显式声明列与字段的对应关系

**参考文件：** `src/main/java/{package}/mapper/`、`src/main/resources/mapper/`

**反模式：** 在 Mapper 接口中写 default 方法实现业务逻辑
