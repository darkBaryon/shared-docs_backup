# 前端页面到后端接口映射

这份文档按“小程序页面/用户动作”来整理接口，方便前端在 `request.ts` 与 `api/endpoint` 中统一封装调用。

约定：
- 除登录相关接口外，其余接口默认都需要 `Authorization: Bearer <token>`
- 后端统一返回 `{ code, message, data }`
- 页面代码不直接拼 `/api/v1/...`，建议通过 `api/endpoint` 暴露语义化方法名

## 推荐的前端 endpoint 分组

可以按下面的语义命名来组织前端 `api/endpoint`：

- `authApi.wechatLogin`
- `chatApi.send`
- `houseApi.search`
- `houseApi.publicDetail`
- `houseApi.create`
- `houseApi.listMine`
- `houseApi.detailMine`
- `houseApi.update`
- `houseApi.updateStatus`
- `houseApi.remove`
- `favoriteApi.add`
- `favoriteApi.remove`
- `favoriteApi.list`
- `historyApi.add`
- `historyApi.list`
- `userApi.profile`
- `userApi.switchRole`
- `userApi.dashboard`

## 1. 小程序启动与登录

页面：
- 小程序启动
- 登录态恢复

调用顺序：
1. 前端执行 `wx.login()`
2. 把 `code` 发送给后端 `POST /api/v1/auth/wechat_login`
3. 后端返回 `token`
4. 前端把 `token` 存本地，后续请求统一走 `request.ts` 自动带上

请求：

```json
{
  "code": "wx-login-code"
}
```

返回重点：

```json
{
  "code": 1,
  "message": "ok",
  "data": {
    "token": "..."
  }
}
```

## 2. 首页找房列表

页面：
- 首页推荐房源
- 找房筛选页
- 房源列表页

接口：
- `POST /api/v1/house/search`

请求体：

```json
{
  "area": "南山",
  "min_price": 3000,
  "max_price": 8000,
  "type": "",
  "tags": [],
  "keyword": "",
  "page": 1,
  "limit": 20
}
```

返回重点：
- `data.total`
- `data.page`
- `data.limit`
- `data.list`

适合前端 endpoint：
- `houseApi.search(params)`

## 3. 房源详情页

页面：
- 租客查看房源详情

接口：
- `POST /api/v1/house/public_detail`

作用：
- 返回租客可浏览的房源详情
- 同时返回当前用户是否已经收藏，前端可以直接决定按钮状态

请求体：

```json
{
  "house_id": "..."
}
```

返回重点：

```json
{
  "code": 1,
  "message": "ok",
  "data": {
    "house": {},
    "is_favorited": true
  }
}
```

前端交互建议：
- 页面加载时先调用这个接口
- 根据 `is_favorited` 渲染“收藏”或“取消收藏”按钮
- 页面进入成功后，可以异步调用一次 `history/add`

适合前端 endpoint：
- `houseApi.publicDetail(houseId)`

## 4. 收藏按钮与收藏列表

页面：
- 房源详情页收藏按钮
- 我的收藏页

### 4.1 添加收藏

接口：
- `POST /api/v1/favorite/add`

请求体：

```json
{
  "house_id": "..."
}
```

适合前端 endpoint：
- `favoriteApi.add(houseId)`

### 4.2 取消收藏

接口：
- `POST /api/v1/favorite/remove`

请求体：

```json
{
  "house_id": "..."
}
```

适合前端 endpoint：
- `favoriteApi.remove(houseId)`

### 4.3 收藏列表

接口：
- `POST /api/v1/favorite/list`

请求体：

```json
{
  "limit": 50
}
```

返回重点：
- `data.favorites`

适合前端 endpoint：
- `favoriteApi.list(limit)`

推荐前端交互：
1. 详情页先调 `house/public_detail`
2. 用户点击收藏时调 `favorite/add`
3. 用户点击取消收藏时调 `favorite/remove`
4. 操作成功后，本地切换 `isFavorited` UI 状态

## 5. 浏览历史

页面：
- 房源详情页自动记录
- 最近浏览页

### 5.1 记录浏览

接口：
- `POST /api/v1/history/add`

请求体：

```json
{
  "house_id": "...",
  "source": "detail"
}
```

适合前端 endpoint：
- `historyApi.add(houseId, source)`

建议：
- 进入房源详情页成功后调用
- 不建议把它绑在按钮点击上，避免漏记

### 5.2 浏览历史列表

接口：
- `POST /api/v1/history/list`

请求体：

```json
{
  "limit": 50
}
```

返回重点：
- `data.history`

适合前端 endpoint：
- `historyApi.list(limit)`

## 6. AI 对话找房

页面：
- AI 找房聊天页

接口：
- `POST /api/v1/chat/send`

请求体：

```json
{
  "message": "我想找南山 5000 左右的一室一厅"
}
```

返回重点：
- `data.user`
- `data.ai`

适合前端 endpoint：
- `chatApi.send(message)`

## 7. 我的主页

页面：
- 我的资料页
- 我的主页统计卡片
- 角色切换入口

### 7.1 当前用户资料

接口：
- `POST /api/v1/user/profile`

请求体：
- 无业务字段，只带 token

返回重点：
- `data.profile`
- `data.auth`

适合前端 endpoint：
- `userApi.profile()`

### 7.2 用户主页统计

接口：
- `POST /api/v1/user/dashboard`

请求体：
- 无业务字段，只带 token

返回重点：
- `data.dashboard.favorite_count`
- `data.dashboard.history_count`
- `data.dashboard.session_count`
- `data.dashboard.house_total`
- `data.dashboard.house_on_rent`
- `data.dashboard.house_rented`
- `data.dashboard.house_offline`

适合前端 endpoint：
- `userApi.dashboard()`

### 7.3 切换角色

接口：
- `POST /api/v1/user/switch_role`

请求体：

```json
{
  "role": "landlord"
}
```

适合前端 endpoint：
- `userApi.switchRole(role)`

## 8. 房东房源管理

页面：
- 我的房源列表
- 创建房源
- 编辑房源
- 修改房源状态

### 8.1 我的房源列表

接口：
- `POST /api/v1/house/list`

请求体：

```json
{
  "status": 1
}
```

适合前端 endpoint：
- `houseApi.listMine(status)`

### 8.2 我的房源详情

接口：
- `POST /api/v1/house/detail`

请求体：

```json
{
  "house_id": "..."
}
```

适合前端 endpoint：
- `houseApi.detailMine(houseId)`

### 8.3 创建房源

接口：
- `POST /api/v1/house/create`

适合前端 endpoint：
- `houseApi.create(payload)`

说明：
- 目前房源字段还在继续往 V2 结构演进
- 前端和后端需要以最新 schema 为准，不要只参考旧 README

### 8.4 修改房源

接口：
- `POST /api/v1/house/update`

适合前端 endpoint：
- `houseApi.update(payload)`

### 8.5 修改房源状态

接口：
- `POST /api/v1/house/status`

适合前端 endpoint：
- `houseApi.updateStatus(houseId, status)`

### 8.6 删除房源

接口：
- `POST /api/v1/house/delete`

适合前端 endpoint：
- `houseApi.remove(houseId)`

## 9. 一个典型页面调用链

### 租客打开房源详情页

调用顺序：
1. `houseApi.publicDetail(houseId)`
2. 页面渲染详情和 `isFavorited`
3. 异步调用 `historyApi.add(houseId, "detail")`
4. 用户点击收藏时：
   - 未收藏 -> `favoriteApi.add(houseId)`
   - 已收藏 -> `favoriteApi.remove(houseId)`

### 用户进入“我的”页面

调用顺序：
1. `userApi.profile()`
2. `userApi.dashboard()`

### 房东进入“我的房源”

调用顺序：
1. `userApi.profile()` 判断当前角色
2. `houseApi.listMine(status)`
3. 点击某条房源再调用 `houseApi.detailMine(houseId)`

## 10. 建议的前端文件组织

如果你们现在已经有 `request.ts` 和 `api/endpoint`，可以继续沿用，只要把命名收拢成业务语义即可。

建议结构：

```text
src/
  api/
    request.ts
    endpoint/
      auth.ts
      chat.ts
      house.ts
      favorite.ts
      history.ts
      user.ts
```

核心原则：
- `request.ts` 负责 token、baseURL、错误处理
- `endpoint/*.ts` 负责 URL 与请求方法
- 页面代码只调封装好的函数，不直接写裸 URL
