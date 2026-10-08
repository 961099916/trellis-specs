# Git 规范

> 来源：~/.htcode/skills/code-standards/_languages/git.md

## 分支

| 分支 | 来源 | 用途 | 命名 |
|---|---|---|---|
| 主干 | — | 可发布状态 | `master` / `main` |
| 开发 | master | 集成分支 | `develop` |
| 特性 | develop | 新功能 | `feature/{需求简称}` |
| 修复 | develop | 非紧急 Bug | `bugfix/{问题简称}` |
| 热修 | master | 线上紧急 | `hotfix/{问题简称}` |
| 发布 | develop | 发布准备 | `release/{版本号}` |

- **禁止直接在 master/develop 上开发**。
- 分支命名用小写中划线，含需求关键词（便于回溯）。
- 合入主干一律走 **Merge Request + 代码评审**，禁止直推。

## Commit Message

```
<type>(<scope>): <subject>

<body>（可选，说明 why）

<footer>（可选，关联需求/工单号）
```

**type 取值**：

| type | 用途 |
|---|---|
| `feat` | 新功能 |
| `fix` | 修 Bug |
| `refactor` | 重构（不改变外部行为） |
| `perf` | 性能优化 |
| `test` | 测试相关 |
| `docs` | 文档 |
| `chore` | 构建/依赖/配置 |
| `revert` | 回滚 |

示例：
```
feat(order): 支持 VIP 权益批量查询

原单条查询在批量场景下单页耗时超 3s，改为批量接口 + 缓存。
关联 JIRA: HSMP-1234
```

规则：
- subject **中文、祈使句、不加句号、≤ 50 字**。
- 一个 commit 只做一件事；`fix` 与 `refactor` 不要混在一个 commit。
- **禁止** `update`、`修改`、`临时提交`、`wip` 这类无信息量 message。

## 提交前检查

```bash
git diff --stat                    # 确认改动范围，别把无关文件带上
git status                         # 无未跟踪的临时文件
```

必查：
- 无调试代码（`System.out`、`printStackTrace`、临时 main 方法）。
- 无注释掉的代码块。
- 无 TODO 无责任人。
- 无本地配置/密钥/密码入库。
- 涉及 SQL 的已跑 `check-sql.sh`。

## 敏感信息

- **任何情况下不提交**密码、密钥、token、内网地址清单到仓库。
- 误提交后：立即轮换凭据（改密码/吊销 token），**不要只删文件**——历史记录里仍在。

## 合并与回滚

- 合入前先 `rebase` 到目标分支最新，解决冲突后自测一遍。
- 冲突解决**必须找原作者确认**，不要凭猜测覆盖。
- 回滚优先用 `git revert`（保留历史），**禁止对已推送的公共分支 `reset --hard` + 强推**。

## AI 红线

- **不代用户执行不可逆 git 操作**：`reset --hard`、`push --force`、`clean -fd`、删除分支。
- 需要时**只给命令 + 说明后果**，由用户执行。
- 提交前展示 `git diff` 让用户确认，不要默认代签。
