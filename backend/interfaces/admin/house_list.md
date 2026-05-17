# POST /api/v1/house/list

## 1. 当前状态

该接口进入 admin MVP Phase 3 `house` 模块第一批实现。

目标：

- 后台员工查看平台全量房源列表。
- 支持运营常用筛选。
- 第一阶段只读，不改变房源状态。

## 2. 入口

- 鉴权：需要 Bearer token。
- 权限：需要 `house.view`。
- handler：`internal/handler/v1/admin/house.(*Handler).List`
- service：`internal/service/admin/house.(*Service).List`
- 主读集合：`hs_hpd_admin_listing`

## 3. 请求体

```json
{
  "provider_id": "landlord_object_id",
  "asset_mode": "centralized",
  "city": "深圳",
  "district": "南山",
  "room_status": 1,
  "listing_status": 3,
  "audit_status": 1,
  "page": 1,
  "page_size": 20
}
```

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `provider_id` | string | 否 | 发房方 ObjectID hex |
| `asset_mode` | string | 否 | `centralized` / `decentralized` |
| `city` | string | 否 | 城市，完整匹配 |
| `district` | string | 否 | 区域，完整匹配 |
| `room_status` | int | 否 | HMD 房态 |
| `listing_status` | int | 否 | HPD 发布状态 |
| `audit_status` | int | 否 | 最近审核状态；`0` 表示无审核单 / 未指定 |
| `page` | int | 否 | 页码，默认 `1` |
| `page_size` | int | 否 | 每页数量，默认 `20`，最大 `100` |

## 4. 响应体

```json
{
  "list": [
    {
      "listing_id": "listing_object_id",
      "source_type": "centralized_room",
      "source_id": "source_object_id",
      "asset_mode": "centralized",
      "provider_id": "landlord_object_id",
      "provider_phone": "13800000000",
      "provider_name": "",
      "project_name": "泊寓南山科技园",
      "building_name": "A座",
      "room_type_name": "一居室",
      "community_name": "",
      "city": "深圳",
      "district": "南山",
      "room_no": "1201",
      "title": "泊寓南山科技园 A座 1201",
      "price": 5800,
      "price_text": "5800元/月",
      "layout_text": "一居室",
      "area_size": 35,
      "room_status": 1,
      "listing_status": 3,
      "audit_status": 2,
      "is_online": 1,
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
| 缺少 `house.view` 权限 | `Forbidden` | `当前账号无权查看房源列表` |
| 请求体格式错误 | `InvalidParam` | `请求参数格式不正确` |
| 参数格式错误 | `InvalidParam` | `房源筛选参数不正确` |
| 查询失败 | `DatabaseError` | `获取房源列表失败，请稍后重试` |

## 6. 关键约束

- 当前接口只读。
- 当前接口不直接聚合 HMD / HAC。
- 当前接口不返回联系人完整信息。
