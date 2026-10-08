# Spring Boot + MyBatis + MySQL 编码规范

本规范面向 Spring Boot + MyBatis + MySQL 技术栈的 Java 后端项目。所有规则均源自本仓库真实代码，AI 写代码前先读本规范。

## 目录结构

```
backend/
  ├── index.md          # 本层导航
  ├── layers.md         # 分层架构约定
  ├── naming.md         # 命名规范
  ├── mybatis.md        # MyBatis 使用约定
  ├── exception.md      # 异常处理规范
  ├── api.md            # API 设计规范
  └── verification.md   # 验证命令

guides/
  ├── index.md
  └── database-migration.md   # 数据库迁移指南
```

## 快速原则

1. Controller 只做参数校验和调用 Service，禁止写业务逻辑
2. Service 层处理所有业务逻辑，事务边界在 Service 层
3. Mapper（DAO）层只做数据库 CRUD，不允许有任何业务判断
4. 数据库表名用下划线命名，Java 字段用驼峰命名，Mapper XML 中配置 resultMap 自动映射
5. 所有对外接口返回统一 JSON 格式 `{ code, message, data }`，禁止直接返回实体
6. 禁止 SQL 字符串拼接，全部使用 MyBatis 参数化查询
7. 日志使用 Slf4j，禁止 `System.out` 和 `e.printStackTrace()`
