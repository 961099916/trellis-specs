# Spring Boot + MyBatis + MySQL 编码规范

本规范面向 Spring Boot + MyBatis + MySQL 技术栈的 Java 后端项目。
**所有规则均源自你的真实代码经验，AI 写代码前先读本规范。**

> 来源：~/.htcode/skills/code-standards/

## 目录结构

```
spring-boot-java/
  ├── java/          # Java 编码规范
  ├── spring-boot/   # Spring Boot 规范
  ├── mybatis/       # MyBatis / SQL 规范
  ├── git/           # Git 规范
  ├── testing/       # 单元测试规范
  └── guides/        # 开发指南
```

## 快速原则

1. Controller 只做参数校验和调用 Service，禁止写业务逻辑
2. Service 层处理所有业务逻辑，事务边界在 Service 层
3. Mapper（DAO）层只做数据库 CRUD，不允许有任何业务判断
4. 所有对外接口返回统一 JSON 格式 `{ code, message, data }`
5. 禁止 SQL 字符串拼接，全部使用 MyBatis 参数化查询
6. 日志使用 Slf4j，禁止 `System.out` 和 `e.printStackTrace()`
7. 一个 commit 只做一件事，提交前必须跑 `check-sql.sh`（涉及 SQL 时）
