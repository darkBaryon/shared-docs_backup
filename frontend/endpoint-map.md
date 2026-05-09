# Frontend Endpoint Contract Map

按页面动作约定接口分组，前端建议以语义化 endpoint 封装。

## auth

- `authApi.login` -> `POST /api/v1/auth/login`
- `authApi.wechatLogin` -> `POST /api/v1/auth/wechat_login`

## house

- `houseApi.search` -> `POST /api/v1/house/search`
- `houseApi.publicDetail` -> `POST /api/v1/house/public_detail`
- `houseApi.create` -> `POST /api/v1/house/create`
- `houseApi.listMine` -> `POST /api/v1/house/list`
- `houseApi.detailMine` -> `POST /api/v1/house/detail`
- `houseApi.update` -> `POST /api/v1/house/update`
- `houseApi.updateStatus` -> `POST /api/v1/house/status`
- `houseApi.remove` -> `POST /api/v1/house/delete`

## favorite

- `favoriteApi.add` -> `POST /api/v1/favorite/add`
- `favoriteApi.remove` -> `POST /api/v1/favorite/remove`
- `favoriteApi.list` -> `POST /api/v1/favorite/list`

## history

- `historyApi.add` -> `POST /api/v1/history/add`
- `historyApi.list` -> `POST /api/v1/history/list`

## chat

- `chatApi.send` -> `POST /api/v1/chat/send`

## user

- `userApi.profile` -> `POST /api/v1/user/profile`
- `userApi.dashboard` -> `POST /api/v1/user/dashboard`
- `userApi.switchRole` -> `POST /api/v1/user/switch_role`

## 通用约定

- 除登录接口外，默认走 `Authorization: Bearer <token>`。
- Go 后端响应统一 `{ code, error, data }`。
- HTTP 非 2xx 优先读取 `error`。
