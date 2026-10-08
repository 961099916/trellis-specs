# 开发指南

## 指南列表

- [database-migration.md](database-migration.md) — Flyway 数据库迁移规范

## 规则

1. 所有数据库变更必须通过 Flyway 迁移脚本，禁止手动直接修改数据库
2. 迁移脚本一旦提交禁止修改，必须通过新脚本修正
3. 每次部署前在本地环境验证迁移脚本
