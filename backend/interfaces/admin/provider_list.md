# POST /api/v1/provider/list

## 1. 当前状态

该接口进入 admin Phase 2 `provider` 模块第一批实现。

目标：

- 后台员工查看发房方账号列表。
- 支持按手机号和状态筛选。
- 返回账号主体和登录摘要。

## 2. 入口

- 鉴权：需要 Bearer token。
- 权限：需要 `provider.view`。
- handler：`internal/handler/v1/admin/provider.(*Handler).List`
- service：`internal/service/admin/provider.(*Service).List`

## 3. 请求体

```json
{
  "phone": "13800000000",
  "status": 1,
  "page": 1,
  "page_size": 20
}
```

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `phone` | string | 否 | 发房方手机号，完整匹配 |
| `status` | int | 否 | 状态；`1` 有效，`-1` 禁用；不传默认 `1` |
| `page` | int | 否 | 页码，默认 `1` |
| `page_size` | int | 否 | 每页数量，默认 `20`，最大 `100` |

## 4. 响应体

```json
{
  "list": [
    {
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
  ],
  "page": 1,
  "page_size": 20,
  "total": 1
}
```

## 5. 处理逻辑

1. auth middleware 校验 admin Bearer token，并注入 `session.Principal`。
2. 权限 middleware 校验当前 principal 包含 `provider.view`。
3. handler 绑定请求体，空 body 按 `{}` 处理。
4. service 规范化分页和状态。
5. 查询 `hs_lld_landlord`。
6. 批量查询 `hs_lld_auth` 登录摘要。
7. 返回列表。

## 6. 失败返回

| 场景 | 错误码 | 对外中文提示 |
| --- | --- | --- |
| 缺少 `provider.view` 权限 | `Forbidden` | `当前账号无权查看发房方列表` |
| 请求体格式错误 | `InvalidParam` | `请求参数格式不正确` |
| 状态参数错误 | `InvalidParam` | `发房方状态参数不正确` |
| 查询失败 | `DatabaseError` | `获取发房方列表失败，请稍后重试` |

## 7. 关键约束

- 当前阶段不返回名称、城市、图片、品牌字段。
- `provider_id` 等于 `hs_lld_landlord._id`。
