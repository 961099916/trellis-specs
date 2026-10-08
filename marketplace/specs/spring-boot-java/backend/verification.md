# 验证命令

在项目根目录执行以下命令验证代码规范。

## 编译

```bash
# 编译检查
mvn compile

# 跳过测试编译
mvn compile -DskipTests
```

## 测试

```bash
# 运行所有单元测试
mvn test

# 运行指定测试类
mvn test -Dtest=UserServiceTest

# 生成测试覆盖率报告
mvn test jacoco:report
```

## 代码质量

```bash
# SpotBugs 静态检查
mvn spotbugs:check

# Checkstyle 格式检查
mvn checkstyle:check

# Maven Enforcer 依赖冲突检查
mvn enforcer:enforce
```

## MyBatis 检查

```bash
# MyBatis XML 语法验证（需要 mybatis-plus-boot-starter）
mvn mybatis-help:inspect
```

## 完整检查链

```bash
# 本地 CI 等效命令（推送前必须通过）
mvn clean compile test spotbugs:check checkstyle:check
```

## Git Hooks（可选）

建议在 `.git/hooks/` 配置 pre-commit 钩子：

```bash
# 安装 pre-commit 钩子（需 pre-commit 工具）
pre-commit install
```

**参考文件：** `pom.xml`、`checkstyle.xml`、`spotbugs-exclude.xml`
