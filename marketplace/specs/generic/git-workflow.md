# Git 工作流

## 分支命名

| 类型 | 格式 | 示例 |
|------|------|------|
| 特性分支 | `feature/{issue-id}-{简短描述}` | `feature/123-user-login` |
| 修复分支 | `fix/{issue-id}-{简短描述}` | `fix/456-null-pointer` |
| 发布分支 | `release/v{x.y.z}` | `release/v1.2.0` |
| 热修复分支 | `hotfix/{issue-id}-{简短描述}` | `hotfix/789-critical-bug` |

## 提交规范

提交信息格式：

```
{type}({scope}): {description}

[optional body]

[optional footer]
```

类型（type）：

- `feat` — 新功能
- `fix` — Bug 修复
- `docs` — 文档更新
- `style` — 代码格式（不影响功能）
- `refactor` — 重构
- `test` — 测试相关
- `chore` — 构建/工具变更

示例：

```
feat(user): 新增用户注册接口

- 支持手机号注册
- 发送验证码前校验格式
- 60 秒防刷限制

Closes #123
```

## 规则

1. **每条提交描述一个原子改动**，禁止混合多个无关改动
2. **禁止 `git push --force` 到 main/master 分支**
3. **PR/MR 必须通过 CI 检查**（测试、格式检查）后才能合并
4. **合并使用 Squash Merge** 保持分支历史整洁
5. **删除已合并的分支**，保持仓库清洁
