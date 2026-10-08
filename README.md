# Trellis Spec 模板库

收录适合中国开发者的 Trellis 编码规范模板。

## 模板列表

### spring-boot-java
**Spring Boot + MyBatis + MySQL 编码规范**

内容源自多年 Java 后端实战经验，包含：
- `java/` — 分层职责、命名、复杂度控制、空值判等、异常、日志、并发事务、集合流、注释、日期金额
- `spring-boot/` — Controller 规范、参数校验、全局异常、事务、配置、DI、幂等、日志链路
- `mybatis/` — 参数绑定、字段规范、WHERE/索引/性能、动态 SQL、批量操作、结果映射、事务锁
- `git/` — 分支策略、Commit Message、提交前检查、敏感信息、合并回滚、AI 红线
- `testing/` — Given-When-Then、断言规范、Mock 边界、边界清单、Fixture 模式
- `guides/` — Flyway 数据库迁移规范

### java-se
Java SE 基础规范，适合纯 Java 库或工具类项目。

### generic
通用模板，适用于任何语言/框架项目，作为起点自行裁剪。

## 安装方法

```bash
trellis init --registry https://github.com/961099916/trellis-specs
```

Trellis 会自动读取 `marketplace/index.json`，选择要安装的模板即可。
