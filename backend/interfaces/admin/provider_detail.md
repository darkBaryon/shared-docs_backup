# POST /api/v1/provider/detail

## 1. 当前状态

该接口进入 admin Phase 2 `provider` 模块第一批实现。

目标：

- 后台员工查看单个发房方账号详情。
- 返回账号主体和认证摘要。

## 2. 入口

- 鉴权：需要 Bearer token。
- 权限：需要 `provider.view`。
- handler：`internal/handler/v1/admin/provider.(*Handler).Detail`
- service：`internal/service/admin/provider.(*Service).Detail`

## 3. 请求体

```json
{
  "provider_id": "landlord_object_id"
}
```

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `provider_id` | string | 是 | 发房方 ObjectID hex，等于 `hs_lld_landlord._id` |

## 4. 响应体

```json
{
  "provider": {
    "provider_id": "landlord_object_id",
    "phone": "13800000000",
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

## 5. 失败返回

| 场景 | 错误码 | 对外中文提示 |
| --- | --- | --- |
| 缺少 `provider.view` 权限 | `Forbidden` | `当前账号无权查看发房方详情` |
| 请求体格式错误 | `InvalidParam` | `请求参数格式不正确` |
| `provider_id` 缺失或格式错误 | `InvalidParam` | `发房方参数不正确` |
| 发房方不存在 | `NotFound` | `发房方不存在或已删除` |
| 查询失败 | `DatabaseError` | `获取发房方详情失败，请稍后重试` |

## 6. 关键约束

- 详情不得返回 `password_hash`。
- 当前阶段不读取 `hs_lld_profile`。
