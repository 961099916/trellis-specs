# API 设计规范

## RESTful 风格

| 操作 | HTTP 方法 | URL 规范 | 示例 |
|------|-----------|----------|------|
| 新增 | POST | `/资源名` | `POST /users` |
| 查询单个 | GET | `/资源名/{id}` | `GET /users/123` |
| 查询列表 | GET | `/资源名` | `GET /users?page=1&size=20` |
| 更新 | PUT | `/资源名/{id}` | `PUT /users/123` |
| 删除 | DELETE | `/资源名/{id}` | `DELETE /users/123` |

## 请求规范

1. **分页参数**：`page`（从 1 开始）、`size`（默认 20，最大 100）
2. **排序参数**：`sort` 字段用逗号分隔，降序加 `-` 前缀：`sort=-createdAt,name`
3. **查询参数**：按字段名传递，多个字段用 AND 语义
4. **请求体**：POST / PUT 使用 JSON body，Header 必须设置 `Content-Type: application/json`

## 响应规范

1. **列表响应**必须包含分页信息：

```json
{
  "code": 200,
  "message": "success",
  "data": {
    "list": [...],
    "total": 100,
    "page": 1,
    "size": 20
  }
}
```

2. **新增/更新后返回完整对象**，禁止只返回成功标志
3. **删除操作**返回成功即可，不返回数据

## 版本控制

API 升级不兼容时，通过 URL 路径版本化：

```
/api/v1/users
/api/v2/users
```

## 规则

1. 禁止在 URL 中使用动词（`/getUser`、`/saveOrder` 都是错误的）
2. 禁止 GET 请求带请求体
3. 敏感参数（密码、Token）禁止出现在 URL query string 中
4. 日期参数使用 ISO 8601 格式：`2026-01-01` 或 `2026-01-01T00:00:00+08:00`

**参考文件：** `src/main/java/{package}/controller/`
