# POST /api/v1/provider/create

## 1. 当前状态

该接口进入 admin Phase 2 `provider` 模块第一批实现。

目标：

- 后台有权限员工创建可登录 publish 端的发房方账号。
- 同时写入发房方主体和密码认证信息。

## 2. 入口

- 鉴权：需要 Bearer token。
- 权限：需要 `provider.edit`。
- handler：`internal/handler/v1/admin/provider.(*Handler).Create`
- service：`internal/service/admin/provider.(*Service).Create`

## 3. 请求体

```json
{
  "phone": "13800000000",
  "password": "123456"
}
```

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `phone` | string | 是 | 发房方登录手机号，全局唯一 |
| `password` | string | 是 | 初始密码，后端 bcrypt hash 后写入 |

## 4. 响应体

```json
{
  "provider": {
    "provider_id": "landlord_object_id",
    "phone": "13800000000",
    "status": 1,
    "created_by_staff_id": "staff_object_id",
    "updated_by_staff_id": "",
    "created_at": 1760000000,
    "updated_at": 1760000000,
    "password_updated_at": 1760000000,
    "last_login_at": 0,
    "last_login_ip": ""
  }
}
```

## 5. 处理逻辑

1. 权限 middleware 校验当前 principal 包含 `provider.edit`。
2. service 校验手机号和密码。
3. 查询 `hs_lld_landlord`，确认手机号未被占用。
4. 写入 `hs_lld_landlord`。
5. 写入 `hs_lld_auth`。
6. 返回发房方账号摘要。

非事务约定：

- 当前开发阶段不启用 Mongo 多集合事务。
- 写入主体后，如果认证信息写入失败，后端清理本次创建的主体和认证信息。

## 6. 失败返回

| 场景 | 错误码 | 对外中文提示 |
| --- | --- | --- |
| 缺少 `provider.edit` 权限 | `Forbidden` | `当前账号无权创建发房方` |
| 请求体格式错误 | `InvalidParam` | `请求参数格式不正确` |
| 必填缺失 | `InvalidParam` | `请输入发房方手机号和初始密码` |
| 手机号格式错误 | `InvalidParam` | `请输入正确的发房方手机号` |
| 手机号重复 | `AlreadyExists` | `发房方手机号已存在，请更换后重试` |
| 创建失败 | `DatabaseError` | `创建发房方失败，请稍后重试` |

## 7. 关键约束

- 该接口不写 `hs_lld_profile`。
- 响应不返回密码或密码 hash。
