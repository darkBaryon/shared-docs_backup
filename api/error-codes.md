# Error Contract

## 统一响应包裹

后端联调以统一结构为准：

```json
{
  "code": 0,
  "message": "ok",
  "data": {}
}
```

- `code = 0`：业务成功。
- 非 2xx HTTP：通常为接口或业务失败，优先读取 `detail`，其次读取 `message`。

## 当前错误 detail 现状

- 当前项目里，很多接口仍直接返回字符串 `detail`，例如 `"房源不存在"`、`"切换角色失败"`。
- 结构化错误对象还未全量落地，前端需要同时兼容字符串 `detail` 与未来可能出现的对象 `detail`。

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
