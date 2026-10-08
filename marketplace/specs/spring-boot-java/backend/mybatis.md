# MyBatis 使用约定

## XML 规范

### 基础规则

- 每个 Mapper XML 文件不超过 300 行
- SQL 关键词大写：`SELECT`、`FROM`、`WHERE`
- 每个标签独占一行，便于版本对比
- 使用 `<![CDATA[ ... ]]>` 包裹含 `<`、`>` 的表达式

```xml
<select id="selectById" resultMap="BaseResultMap">
  SELECT <include refid="Base_Column_List"/>
  FROM user_account
  WHERE id = #{id}
</select>
```

### resultMap 配置

- 所有查询必须使用 `resultMap`，禁止使用 `resultType="Map"`
- association 和 collection 使用 `select` 嵌套查询时，必须指定 `columnPrefix`
- 禁止在 resultMap 中省略任何列

```xml
<resultMap id="BaseResultMap" type="com.example.UserAccount">
  <id column="id" property="id"/>
  <result column="username" property="username"/>
  <result column="created_at" property="createdAt"/>
</resultMap>
```

## SQL 编写规范

1. **禁止 SQL 字符串拼接**，全部使用 `#{param}` 参数化查询
2. **禁止 `SELECT *`**，必须显式列出所需列
3. **禁止在 SQL 中做业务计算**（IFNULL、CASE WHEN 尽量移到 Java 代码）
4. **分页必须使用 `LIMIT` + `OFFSET`**，禁止全表扫描后 Java 层截断
5. **批量操作不超过 1000 条**，超时分批执行
6. **模糊查询必须用 `LIKE CONCAT('%', #{keyword}, '%')`**

**反模式：**
```xml
<!-- 错误 -->
<if test="name != null">
  AND name LIKE '%${name}%'
</if>

<!-- 正确 -->
<if test="name != null">
  AND name LIKE CONCAT('%', #{name}, '%')
</if>
```

## Mapper 接口规范

- 方法参数超过 1 个时，使用 `@Param` 注解显式命名
- 禁止在 Mapper 接口中写 default 方法

```java
// 正确：多参数显式命名
List<User> findByNameAndStatus(
    @Param("name") String name,
    @Param("status") Integer status
);
```

## 动态 SQL

- `<if>` 条件判断字段是否存在，用 `!= null` 而非 `!''.equals(xxx)`
- `<where>` 标签自动处理 AND/OR 前缀，比手写 `WHERE 1=1` 更安全
- `<set>` 标签自动处理 UPDATE 字段后的逗号问题
- 动态批量插入使用 `<foreach>`，collection 类型用 `@Param("list")` 声明

```xml
<insert id="saveBatch" parameterType="java.util.List">
  INSERT INTO user_account (username, phone, status)
  VALUES
  <foreach collection="list" item="item" separator=",">
    (#{item.username}, #{item.phone}, #{item.status})
  </foreach>
</insert>
```
