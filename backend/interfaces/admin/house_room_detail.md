# POST /api/v1/house_room/detail

## 1. 当前状态

该接口用于后台查看单个房间详情摘要。

## 2. 入口

- 鉴权：需要 Bearer token。
- 权限：需要 `house.view`。
- handler：`internal/handler/v1/admin/house.(*Handler).DetailRoom`
- service：`internal/service/admin/house.(*Service).DetailRoom`
- 主读集合：`hs_hpd_admin_listing`

## 3. 请求体

```json
{
  "listing_id": "listing_object_id"
}
```

## 4. 失败返回

| 场景 | 错误码 | 对外中文提示 |
| --- | --- | --- |
| 缺少 `house.view` 权限 | `Forbidden` | `当前账号无权查看房间详情` |
| 请求体格式错误 | `InvalidParam` | `请求参数格式不正确` |
| `listing_id` 缺失或格式错误 | `InvalidParam` | `房源参数不正确` |
| 房源不存在 | `NotFound` | `房源不存在或已删除` |
| 查询失败 | `DatabaseError` | `获取房源详情失败，请稍后重试` |
