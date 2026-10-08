# 数据库迁移规范

## 工具

使用 Flyway 进行数据库版本化管理。

## 规则

### 脚本命名

```
V{版本号}__{描述}.sql
```

- 版本号：4 位数字，如 `V0001__init_user_account.sql`
- 描述：下划线分隔的小写英文，描述本次迁移内容
- 版本号必须全局唯一，禁止重复

### 脚本内容

1. **每条语句单独一行**，便于版本对比
2. **禁止 DROP TABLE**，如需清理数据走软删除
3. **禁止 ALTER COLUMN 类型**（风险过高，评估后通过新表迁移）
4. **新增字段必须有默认值或允许 NULL**，避免历史数据无法写入
5. **新增字段放在末尾**，禁止插入到中间位置

### 示例

```sql
-- V0001__init_user_account.sql
CREATE TABLE IF NOT EXISTS user_account (
    id          BIGINT          NOT NULL AUTO_INCREMENT COMMENT '主键ID',
    username    VARCHAR(50)     NOT NULL COMMENT '用户名',
    phone       VARCHAR(20)     COMMENT '手机号',
    status      TINYINT         NOT NULL DEFAULT 1 COMMENT '状态：0-禁用，1-正常',
    created_at  DATETIME        NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    updated_at  DATETIME        NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
    deleted     TINYINT         NOT NULL DEFAULT 0 COMMENT '软删除标记：0-未删除，1-已删除',
    PRIMARY KEY (id),
    UNIQUE KEY uk_username (username),
    KEY idx_phone (phone)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='用户账户表';
```

### 验证

```bash
# 本地验证迁移
mvn flyway:migrate -Dflyway.cleanDisabled=false

# 验证当前状态
mvn flyway:info
```

## 禁止事项

- 禁止在 SQL 中写业务逻辑（存储过程、触发器）
- 禁止在迁移脚本中插入测试数据
- 禁止 `TRUNCATE TABLE`
