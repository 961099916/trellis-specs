# 通用 Trellis Spec 模板

本模板适用于任何语言、任何框架的项目，作为起点按需裁剪。

## 目录结构

```
generic/
  ├── index.md       # 导航
  ├── project-layout.md    # 项目结构规范
  ├── coding-style.md      # 代码风格
  ├── testing.md           # 测试规范
  └── git-workflow.md      # Git 工作流
```

## 快速原则

1. 每个目录下必须有 `index.md` 导航文件
2. 每条规范必须有具体文件路径作为证据
3. 不适用的章节直接删除，不要留占位文本
4. 规范描述的是**本仓库**的本地模式，不是通用最佳实践

## 使用方法

1. 下载本模板到项目 `.trellis/spec/` 目录
2. 根据项目技术栈替换各章节内容
3. 删除不适用的章节
4. 更新文件路径为项目真实路径
5. 运行 `grep -R "placeholder\|TODO" .trellis/spec` 确认无遗留
