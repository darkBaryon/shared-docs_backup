# Error Contract

## 统一响应包裹

后端联调以统一结构为准：

```json
{
  "code": 0,
  "error": "",
  "data": {}
}
```

- `code = 0`：业务成功。
- 非 2xx HTTP：通常为接口或业务失败，优先读取 `code` 和 `error`。

## 当前错误现状

- Go 后端当前统一返回 `{ code, error, data }`。
- `error` 是面向用户或联调定位的错误信息。
- 更细的错误详情暂未作为稳定对外字段暴露。

## 推荐错误对象（未来目标）

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
