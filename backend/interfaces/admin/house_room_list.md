# POST /api/v1/house_room/list

## 1. 当前状态

该接口用于后台房源管理第三层级的房间列表。

目标：

- 在选定项目/小区后查看房间列表。
- 集中式可继续按 `building_id` 缩小到单楼栋。
- 第一阶段只读，不改变房源状态。

## 2. 入口

- 鉴权：需要 Bearer token。
- 权限：需要 `house.view`。
- handler：`internal/handler/v1/admin/house.(*Handler).ListRooms`
- service：`internal/service/admin/house.(*Service).ListRooms`
- 主读集合：`hs_hpd_admin_listing`

## 3. 请求体

```json
{
  "root_id": "root_object_id",
  "building_id": "building_object_id",
  "room_status": 1,
  "listing_status": 3,
  "audit_status": 1,
  "page": 1,
  "page_size": 20
}
```

## 4. 失败返回

| 场景 | 错误码 | 对外中文提示 |
| --- | --- | --- |
| 缺少 `house.view` 权限 | `Forbidden` | `当前账号无权查看房间列表` |
| 请求体格式错误 | `InvalidParam` | `请求参数格式不正确` |
| `root_id` 缺失或格式错误 | `InvalidParam` | `项目/小区参数不正确` |
| `building_id` 格式错误 | `InvalidParam` | `楼栋参数不正确` |
| 其它筛选参数错误 | `InvalidParam` | `房源筛选参数不正确` |
| 查询失败 | `DatabaseError` | `获取房源列表失败，请稍后重试` |

## 5. 关键约束

- `root_id` 必填，不再支持后台首页直接平铺全量房间。
- 集中式传 `building_id` 时只看单楼栋；分散式不传 `building_id`。
