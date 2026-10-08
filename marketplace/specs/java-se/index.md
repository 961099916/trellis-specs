# Java SE 基础编码规范

本规范适合纯 Java SE 环境下的库、工具类、控制台应用项目。不依赖 Spring 生态，覆盖命名、异常、日志、单元测试等通用场景。

## 目录结构

```
java-se/
  ├── index.md        # 导航
  ├── naming.md       # 命名规范
  ├── exception.md    # 异常处理
  ├── logging.md      # 日志规范
  └── testing.md      # 单元测试规范
```

## 快速原则

1. 所有资源（Stream、Connection、Channel）使用 try-with-resources
2. 集合操作优先使用 Stream API 和 Lambda
3. 日期时间处理使用 `java.time` API，禁止使用 `Date`、`Calendar`
4. 字符串拼接优先使用 `StringBuilder`，循环内拼接禁止使用 `+`
5. 所有公开方法必须有 Javadoc，重要逻辑在方法体内加行内注释
6. 单元测试必须可通过 `mvn test` 无报错执行，测试之间无依赖
