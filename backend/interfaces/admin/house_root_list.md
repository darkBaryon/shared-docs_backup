# POST /api/v1/house_root/list

## 1. 当前状态

该接口用于后台房源管理首页的项目/小区列表。

目标：

- 后台员工按项目/小区查看全平台房源入口。
- 作为房源管理第一层级，不再直接平铺房间。
- 第一阶段只读，不改变房源状态。

## 2. 入口

- 鉴权：需要 Bearer token。
- 权限：需要 `house.view`。
- handler：`internal/handler/v1/admin/house.(*Handler).ListRoots`
- service：`internal/service/admin/house.(*Service).ListRoots`
- 主读集合：`hs_hpd_admin_listing`

## 3. 请求体

```json
{
  "provider_id": "landlord_object_id",
  "asset_mode": "centralized",
  "city": "深圳",
  "district": "南山",
  "page": 1,
  "page_size": 20
}
```

## 4. 响应体

```json
{
  "list": [
    {
      "root_id": "root_object_id",
      "root_type": "centralized_project",
      "root_name": "泊寓南山科技园",
      "asset_mode": "centralized",
      "provider_id": "landlord_object_id",
      "provider_phone": "13800000000",
      "provider_name": "张三",
      "project_id": "project_object_id",
      "project_name": "泊寓南山科技园",
      "community_id": "",
      "community_name": "",
      "city": "深圳",
      "district": "南山",
      "biz_area": "科技园",
      "building_count": 2,
      "room_count": 24,
      "updated_at": 1760000000
    }
  ],
  "page": 1,
  "page_size": 20,
  "total": 1
}
```

## 5. 失败返回

| 场景 | 错误码 | 对外中文提示 |
| --- | --- | --- |
| 缺少 `house.view` 权限 | `Forbidden` | `当前账号无权查看项目/小区列表` |
| 请求体格式错误 | `InvalidParam` | `请求参数格式不正确` |
| 参数格式错误 | `InvalidParam` | `房源筛选参数不正确` |
| 查询失败 | `DatabaseError` | `获取项目/小区列表失败，请稍后重试` |

## 6. 关键约束

- 根对象统一抽象为 `root`，取值只可能是 `centralized_project` 或 `decentralized_community`。
- 接口只读 `hs_hpd_admin_listing`，不直接聚合 HMD 主表。
- 后台房源管理首页展示项目/小区，不再直接展示房间列表。
