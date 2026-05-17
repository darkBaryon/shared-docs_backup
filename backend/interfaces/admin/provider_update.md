# POST /api/v1/provider/update

## 1. 当前状态

该接口进入 admin Phase 2 `provider` 模块第一批实现。

目标：

- 后台有权限员工修改发房方登录手机号。
- 修改后失效该发房方旧 publish token。

## 2. 入口

- 鉴权：需要 Bearer token。
- 权限：需要 `provider.edit`。
- handler：`internal/handler/v1/admin/provider.(*Handler).Update`
- service：`internal/service/admin/provider.(*Service).Update`

## 3. 请求体

```json
{
  "provider_id": "landlord_object_id",
  "phone": "13800000001"
}
```

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `provider_id` | string | 是 | 发房方 ObjectID hex |
| `phone` | string | 是 | 新登录手机号 |

## 4. 响应体

```json
{
  "provider": {
    "provider_id": "landlord_object_id",
    "phone": "13800000001",
    "status": 1,
    "created_by_staff_id": "staff_object_id",
    "updated_by_staff_id": "staff_object_id",
    "created_at": 1760000000,
    "updated_at": 1760000000,
    "password_updated_at": 1760000000,
    "last_login_at": 1760000000,
    "last_login_ip": "127.0.0.1"
  }
}
```

## 5. 处理逻辑

1. 权限 middleware 校验当前 principal 包含 `provider.edit`。
2. service 校验 `provider_id` 和手机号。
3. 查询发房方是否存在。
4. 查询新手机号是否被其它发房方占用。
5. 更新 `hs_lld_landlord.phone` 和 `updated_by_staff_id`。
6. 失效该发房方旧 publish token。
7. 返回最新详情。

## 6. 失败返回

| 场景 | 错误码 | 对外中文提示 |
| --- | --- | --- |
| 缺少 `provider.edit` 权限 | `Forbidden` | `当前账号无权编辑发房方` |
| 请求体格式错误 | `InvalidParam` | `请求参数格式不正确` |
| `provider_id` 缺失或格式错误 | `InvalidParam` | `发房方参数不正确` |
| 手机号格式错误 | `InvalidParam` | `请输入正确的发房方手机号` |
| 手机号重复 | `AlreadyExists` | `发房方手机号已存在，请更换后重试` |
| 发房方不存在 | `NotFound` | `发房方不存在或已删除` |
| 更新失败 | `DatabaseError` | `更新发房方失败，请稍后重试` |

## 7. 关键约束

- 当前阶段只支持修改手机号。
- 不允许在该接口修改密码。
- 不写 `hs_lld_profile`。
