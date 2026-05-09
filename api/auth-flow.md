# Auth & Session Flow

## 登录流程（当前实际）

1. 前端调用 `wx.login()` 获取临时 `code`。
2. 前端请求 `POST /api/v1/auth/wechat_login`，请求体 `{ code }`。
3. Go 后端返回 `{ code, error, data }`，成功时 `data.token` 可用。
4. 前端将 `token` 写入本地安全存储。
5. 后续接口通过 `Authorization: Bearer <token>` 传递身份。

## 失败处理约定

1. 若 HTTP 非 2xx，前端优先读取 `error`。
2. 若 HTTP 2xx 但 `code != 0`，按业务失败处理并提示 `error`。
3. token 失效时，清理本地 token 并重新执行登录流程。
