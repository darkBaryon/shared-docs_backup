# AI House 找房系统后端

AI House 是一个面向租房场景的小程序后端，目标是把“房源管理、公开找房、账号体系、AI 对话咨询”整合成一套可持续演进的服务底座。当前版本已经完成 FastAPI 化改造、MongoDB/Redis 基础设施接入、统一鉴权、微信小程序静默登录，以及面向租客的公开房源搜索能力。

这份 README 同时面向两类读者：
- 老板/产品/项目负责人：快速了解项目当前能力、技术方向、可演示内容。
- 前端/联调同学：直接获取接口路径、请求结构、鉴权方式与联调示例。

## 项目现状

当前后端已经具备以下核心能力：
- 统一账号体系：采用 `hs_usr_user + hs_usr_auth` 双表模型，支持未来扩展微信、支付宝、手机号等多渠道登录。
- 微信小程序登录：支持 `code -> openid -> 系统 token` 的静默登录链路。
- 房东房源管理：支持创建、列表、详情、更新、状态变更、逻辑删除。
- 租客公开查房：支持按区域、预算、户型、标签、关键词分页搜索公开房源。
- AI 找房对话：登录后可通过统一 token 调用聊天接口，结合房源数据进行咨询。
- 存储架构：MongoDB 负责持久化，Redis 负责会话态与登录态缓存。

适合当前阶段的演示路径：
1. 小程序微信静默登录
2. 租客搜索公开房源
3. 房东查看/管理自己的房源
4. 用户带 token 发起 AI 找房对话

## 技术方案

- 服务框架：FastAPI
- 语言版本：Python 3.10+
- 数据库：MongoDB + Motor / PyMongo
- 缓存：Redis
- 配置管理：Pydantic Settings + YAML + 环境变量覆盖
- AI 能力：多场景模型路由（`chat`、`compliance`、`intent`、`summary`）

核心目录：

```text
ai_house/
├── main.py
├── config/config.yaml
├── dev.sh
├── requirements/
├── scripts/
├── src/
│   ├── app.py
│   ├── dependencies.py
│   ├── api/v1/
│   ├── services/
│   ├── repositories/
│   ├── core/
│   └── domain/
└── templates/
```

## 启动方式

### 1. 安装依赖

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -U pip
pip install -r requirements/base.txt
```

开发时也可以安装完整依赖：

```bash
pip install -r requirements/dev.txt
```

### 2. 准备基础设施

如果需要连接公司环境的 MongoDB / Redis，先执行：

```bash
./dev.sh
```

该脚本会将公司 K8s 环境中的 MongoDB 与 Redis 映射到本地：
- MongoDB: `127.0.0.1:27018`
- Redis: `127.0.0.1:6380`

### 3. 配置环境变量

常用环境变量：

```bash
export MONGO_PASS='***'
export REDIS_PASS='***'
export WECHAT_APPID='***'
export WECHAT_SECRET='***'
```

配置读取优先级：
1. 启动参数 `-c /path/to/config.yaml`
2. 环境变量 `CONFIG_PATH`
3. `./config/config.yaml`
4. `/etc/config/config.yaml`

### 4. 启动服务

```bash
python main.py
```

默认访问地址：

```text
http://127.0.0.1:5000
```

## 接口总览

项目接口统一遵循：

```text
POST /api/v1/{module}/{action}
```

当前主要接口：

### 认证

- `POST /api/v1/auth/login`
- `POST /api/v1/auth/wechat_login`

### 聊天

- `POST /api/v1/chat/send`

### 房东房源管理

- `POST /api/v1/house/create`
- `POST /api/v1/house/list`
- `POST /api/v1/house/detail`
- `POST /api/v1/house/update`
- `POST /api/v1/house/status`
- `POST /api/v1/house/delete`

### 租客公开查房

- `POST /api/v1/house/search`

## 统一请求与响应约定

### 鉴权

除登录接口外，其他需要用户身份的接口均通过 Bearer Token 鉴权：

```http
Authorization: Bearer <token>
```

### 统一响应结构

当前后端实际返回格式如下：

```json
{
  "code": 1,
  "message": "ok",
  "data": {}
}
```

说明：
- `code = 1` 表示业务成功
- 非 2xx HTTP 通常表示接口或业务异常，错误信息优先看 `detail`

前端建议统一处理：
- HTTP 2xx：再判断 `code === 1`
- HTTP 非 2xx：优先读取 `detail`，其次读取 `message`

## 前端联调指南

### 1. 微信静默登录

前端先调用微信登录：

```ts
wx.login({
  success(res) {
    console.log(res.code)
  }
})
```

然后将 `code` 发送给后端：

```bash
curl -X POST http://127.0.0.1:5000/api/v1/auth/wechat_login \
  -H "Content-Type: application/json" \
  -d '{
    "code": "wx-login-code"
  }'
```

成功返回：

```json
{
  "code": 1,
  "message": "ok",
  "data": {
    "token": "..."
  }
}
```

前端拿到 `token` 后写入本地缓存即可：

```ts
wx.setStorageSync('token', token)
```

### 2. 租客公开查房

这个接口不强制要求登录，适合首页、列表页、筛选页直接使用。

请求：

```bash
curl -X POST http://127.0.0.1:5000/api/v1/house/search \
  -H "Content-Type: application/json" \
  -d '{
    "area": "南山",
    "min_price": 5000,
    "max_price": 9000,
    "type": "两室一厅",
    "tags": ["近地铁", "精装修"],
    "keyword": "科技园",
    "page": 1,
    "limit": 10
  }'
```

返回：

```json
{
  "code": 1,
  "message": "ok",
  "data": {
    "total": 12,
    "page": 1,
    "limit": 10,
    "list": [
      {
        "house_id": "67fd....",
        "area": "南山",
        "location": "科技园公馆·科技园",
        "type": "两室一厅",
        "price": 7800,
        "tags": ["近地铁", "精装修", "采光好"],
        "images": ["https://..."],
        "status": 1,
        "created_at": 1710000000,
        "updated_at": 1710000000
      }
    ]
  }
}
```

字段说明：
- `total`: 总条数
- `page`: 当前页
- `limit`: 每页条数
- `list`: 房源列表
- `house_id`: 前端后续详情页、收藏、预约等动作可使用的房源标识

筛选规则说明：
- `min_price = 0` 或 `max_price = 0` 视为未指定
- `area = ""` 视为不限区域
- `type = ""` 视为不限户型
- `tags = []` 视为不按标签过滤
- `keyword = ""` 视为不按关键词过滤

### 3. 通用账号登录

该接口主要用于开发期手动指定用户联调，正式小程序优先使用微信登录。

请求：

```bash
curl -X POST http://127.0.0.1:5000/api/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "user_id": "demo_user_001"
  }'
```

### 4. 发起聊天

请求：

```bash
curl -X POST http://127.0.0.1:5000/api/v1/chat/send \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer <token>" \
  -d '{
    "message": "我想在福田租两居室，预算 8000 左右"
  }'
```

返回：

```json
{
  "code": 1,
  "message": "ok",
  "data": {
    "user": {
      "text": "我想在福田租两居室，预算 8000 左右",
      "timestamp": "19:20:00"
    },
    "ai": {
      "text": "好的，我先帮您筛选福田区域预算 8000 左右的两居室房源。",
      "timestamp": "19:20:01"
    }
  }
}
```

### 5. 房东管理房源

这组接口需要登录，并带 Bearer Token。

#### 创建房源

`POST /api/v1/house/create`

请求体：

```json
{
  "area": "宝安",
  "location": "宝中壹城·宝安中心",
  "type": "两室一厅",
  "price": 6800,
  "tags": ["近地铁", "精装修"],
  "images": ["https://example.com/1.jpg"],
  "status": 1
}
```

#### 我的房源列表

`POST /api/v1/house/list`

请求体：

```json
{
  "status": 1
}
```

说明：
- `status` 可为空，表示查询全部
- `1` 表示待租
- `2` 表示已租
- `-1` 表示已删除

#### 房源详情

`POST /api/v1/house/detail`

```json
{
  "house_id": "..."
}
```

#### 更新房源

`POST /api/v1/house/update`

```json
{
  "house_id": "...",
  "price": 7200,
  "tags": ["近地铁", "带阳台"]
}
```

#### 更新房源状态

`POST /api/v1/house/status`

```json
{
  "house_id": "...",
  "status": 2
}
```

#### 删除房源

`POST /api/v1/house/delete`

```json
{
  "house_id": "..."
}
```

## 数据设计摘要

### 用户与授权

采用账号与授权双表模型：

- `hs_usr_user`
  - 用户基础档案
  - 字段示例：`user_id`、`nickname`、`role`、`global_preferences`、`status`、`created_at`、`last_active`

- `hs_usr_auth`
  - 用户登录授权表
  - 字段示例：`user_id`、`identity_type`、`identifier`、`credential`
  - 已支持：微信小程序 `openid`

### 房源

房源集合：

- `hs_hs_house`

核心字段：

- `owner_id`
- `area`
- `location`
- `type`
- `price`
- `tags`
- `images`
- `status`
- `created_at`
- `updated_at`

## 当前演示数据

项目已支持一键清空并生成演示数据，便于联调与演示：

```bash
python scripts/seed_demo_data.py
```

默认生成：
- 20 个用户
- 5 个房东
- 15 个租客
- 20 条授权记录
- 每个房东 10-20 套深圳房源

## 开发建议

常用命令：

```bash
pytest -q
ruff check .
black .
mypy src
```

建议下一阶段优先推进：
- 公开房源详情页接口
- 收藏 / 预约看房接口
- 手机号登录与多账号绑定
- 接口自动化测试与前后端联调文档固化

## 注意事项

- 当前接口统一使用 `POST`
- 当前业务成功码为 `code = 1`
- 微信登录依赖 `WECHAT_APPID`、`WECHAT_SECRET`
- 启动时会检查 MongoDB / Redis 连接，失败会直接中断启动
- 前端如果仍使用旧路径，需要统一迁移到 `/api/v1/...`
