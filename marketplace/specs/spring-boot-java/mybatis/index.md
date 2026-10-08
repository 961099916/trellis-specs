# MyBatis / SQL 规范

> 来源：~/.htcode/skills/code-standards/_languages/mybatis.md

## 参数绑定（硬约束）

```xml
<!-- ✅ 预编译 -->
WHERE order_no = #{orderNo}

<!-- ❌ 拼接，SQL 注入 -->
WHERE order_no = '${orderNo}'
```

`${}` 唯一可接受场景：**ORDER BY 的列名/方向**，且必须走白名单校验：

```java
// 列名白名单，不在名单内直接拒绝
if (!SORTABLE_COLUMNS.contains(sortBy)) sortBy = "create_time";
```

## 查询字段

```xml
<!-- ❌ -->
SELECT * FROM t_order

<!-- ✅ 明确字段 -->
SELECT id, order_no, user_id, status, amount, create_time FROM t_order
```

理由：表结构变更会 silently 改变结果；无用字段浪费带宽；覆盖索引失效。

## WHERE 条件

- 单表 UPDATE/DELETE **必须有 WHERE**，无条件的全表操作一律拒绝。
- 逻辑删除表查询**必须带 `is_deleted = 0`**。
- 时间范围用 `>=` 起始 且 `<` 结束（半开区间），避免边界重复/遗漏。
- 分页必须有 `LIMIT`，且深分页（offset 大）改用**游标分页**：
  ```sql
  -- ❌ 深分页慢
  LIMIT 100000, 20
  -- ✅ 游标
  WHERE id > #{lastId} ORDER BY id LIMIT 20
  ```

## 索引与性能

| 反模式 | 后果 | 修法 |
|---|---|---|
| 索引列上用函数 `DATE(create_time)=?` | 索引失效 | 改写为范围条件 |
| 隐式类型转换（字符串字段传数字） | 索引失效 | 类型保持一致 |
| `LIKE '%xxx'` 前置通配符 | 全表扫描 | 倒排 / 全文索引 |
| `OR` 连接不同列 | 常导致索引失效 | 改 `UNION ALL` 或 IN |
| 子查询嵌套 > 3 层 | 难优化 | 拆成多步或 JOIN |
| 大表 `JOIN` 无驱动表概念 | 慢查询 | 小表驱动大表 |
| `count(*)` 在无限定条件大表上 | 全表扫 | 加条件或走汇总表 |

**JOIN 必须带别名**，字段全部带别名前缀，避免列名歧义：
```sql
SELECT o.id, o.order_no, u.user_name
FROM t_order o
LEFT JOIN t_user u ON o.user_id = u.id AND u.is_deleted = 0
WHERE o.is_deleted = 0
```

## 动态 SQL

```xml
<!-- ✅ 用 <where> 自动处理 AND -->
<select id="list" resultType="Order">
  SELECT id, order_no, status FROM t_order
  <where>
    is_deleted = 0
    <if test="status != null">AND status = #{status}</if>
    <if test="userIds != null and !userIds.isEmpty()">
      AND user_id IN
      <foreach collection="userIds" item="id" open="(" separator="," close=")">#{id}</foreach>
    </if>
  </where>
</select>
```

- `<if>` 判空用 `!= null and !xxx.isEmpty()`，不要用 `xxx != ''`（集合不适用）。
- **IN 列表长度要限制**（建议 ≤ 1000），超量分批。
- 避免 `<choose>` 嵌套过深，超过 3 个分支考虑拆成多个 statement。

## 批量操作

```xml
<!-- ✅ 批量插入，必须分批（每批 ≤ 500） -->
<insert id="batchInsert">
  INSERT INTO t_order (order_no, user_id, status) VALUES
  <foreach collection="list" item="item" separator=",">
    (#{item.orderNo}, #{item.userId}, #{item.status})
  </foreach>
</insert>
```

禁止在循环里逐条 insert（网络往返 + 事务日志膨胀）。

## 结果映射

- 字段名与属性名不一致时用 `resultMap`，不要靠 `AS` 全局改别名掩盖设计问题。
- 一对一用 `<association>`，一对多用 `<collection>`；**一对多注意分页会被子表放大**（先查主表 ID 再查子表）。
- 大字段（TEXT/BLOB）单独映射，列表查询不要带。

## 事务与锁

- 更新用**条件原子更新**防并发覆盖，判断影响行数。
- 悲观锁 `SELECT ... FOR UPDATE` 必须在事务内，且**锁顺序全局一致**，否则死锁。
- 乐观锁用 `version` 字段：`UPDATE ... SET version = version + 1 WHERE id = ? AND version = ?`。

## 检查脚本

涉及 SQL / Mapper 改动时，提交前必跑：

```bash
~/.htcode/skills/code-standards/scripts/check-sql.sh <文件或目录>
```
