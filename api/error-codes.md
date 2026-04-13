# Error Contract

## 统一响应包裹

后端联调以统一结构为准：

```json
{
  "code": 1,
  "message": "ok",
  "data": {}
}
```

- `code = 1`：业务成功。
- 非 2xx HTTP：通常为接口或业务失败，优先读取 `detail`，其次读取 `message`。

## 推荐错误对象（detail）

```json
{
  "code": "INVALID_INPUT",
  "message": "Invalid email",
  "details": {}
}
```

## 错误码枚举

- `INVALID_INPUT`：请求参数校验失败。
- `UNAUTHORIZED`：未登录、token 无效或权限不足。
- `NOT_FOUND`：目标资源不存在。
- `INTERNAL_ERROR`：服务内部错误。
