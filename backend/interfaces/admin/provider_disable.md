# POST /api/v1/provider/disable

## 1. 当前状态

该接口进入 admin Phase 2 `provider` 模块第一批实现。

目标：

- 后台有权限员工禁用发房方账号。
- 禁用后发房方不能登录 publish 端。
- 禁用后旧 publish token 失效。

## 2. 入口

- 鉴权：需要 Bearer token。
- 权限：需要 `provider.edit`。
- handler：`internal/handler/v1/admin/provider.(*Handler).Disable`
- service：`internal/service/admin/provider.(*Service).Disable`

## 3. 请求体

```json
{
  "provider_id": "landlord_object_id"
}
```

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `provider_id` | string | 是 | 发房方 ObjectID hex |

## 4. 响应体

```json
{
  "success": true
}
```

## 5. 处理逻辑

1. 权限 middleware 校验当前 principal 包含 `provider.edit`。
2. service 校验 `provider_id`。
3. 查询发房方是否存在。
4. 更新 `hs_lld_landlord.status=-1` 和 `updated_by_staff_id`。
5. 失效该发房方旧 publish token。
6. 返回 `success=true`。

## 6. 失败返回

| 场景 | 错误码 | 对外中文提示 |
| --- | --- | --- |
| 缺少 `provider.edit` 权限 | `Forbidden` | `当前账号无权禁用发房方` |
| 请求体格式错误 | `InvalidParam` | `请求参数格式不正确` |
| `provider_id` 缺失或格式错误 | `InvalidParam` | `发房方参数不正确` |
| 发房方不存在 | `NotFound` | `发房方不存在或已删除` |
| 禁用失败 | `DatabaseError` | `禁用发房方失败，请稍后重试` |

## 7. 关键约束

- 禁用只更新状态，不物理删除。
- 禁用后新登录会被拒绝，旧 publish token 也必须失效。
