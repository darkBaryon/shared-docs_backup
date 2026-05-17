# POST /api/v1/house/detail

## 1. 当前状态

该接口进入 admin MVP Phase 3 `house` 模块第一批实现。

目标：

- 后台员工查看单个房源详情摘要。
- 第一阶段只读，不改变房源状态。

## 2. 入口

- 鉴权：需要 Bearer token。
- 权限：需要 `house.view`。
- handler：`internal/handler/v1/admin/house.(*Handler).Detail`
- service：`internal/service/admin/house.(*Service).Detail`
- 主读集合：`hs_hpd_admin_listing`

## 3. 请求体

```json
{
  "listing_id": "listing_object_id"
}
```

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `listing_id` | string | 是 | HPD listing ObjectID hex |

## 4. 响应体

```json
{
  "house": {
    "listing_id": "listing_object_id",
    "source_type": "centralized_room",
    "source_id": "source_object_id",
    "asset_mode": "centralized",
    "provider_id": "landlord_object_id",
    "provider_phone": "13800000000",
    "provider_name": "",
    "root_type": "centralized_project",
    "root_id": "project_object_id",
    "project_id": "project_object_id",
    "project_name": "泊寓南山科技园",
    "building_id": "building_object_id",
    "building_name": "A座",
    "room_type_id": "room_type_object_id",
    "room_type_name": "一居室",
    "decentralized_id": "",
    "community_name": "",
    "rent_mode": "whole",
    "city": "深圳",
    "district": "南山",
    "biz_area": "",
    "address_text": "深圳市南山区...",
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
    "latest_audit_task_id": "audit_task_object_id",
    "latest_submitted_at": 1760000000,
    "latest_reviewed_at": 1760000000,
    "reviewer_staff_id": "staff_object_id",
    "created_at": 1760000000,
    "updated_at": 1760000000
  }
}
```

## 5. 失败返回

| 场景 | 错误码 | 对外中文提示 |
| --- | --- | --- |
| 缺少 `house.view` 权限 | `Forbidden` | `当前账号无权查看房源详情` |
| 请求体格式错误 | `InvalidParam` | `请求参数格式不正确` |
| `listing_id` 缺失或格式错误 | `InvalidParam` | `房源参数不正确` |
| 房源不存在 | `NotFound` | `房源不存在或已删除` |
| 查询失败 | `DatabaseError` | `获取房源详情失败，请稍后重试` |

## 6. 关键约束

- 当前接口只读。
- 当前详情是运营摘要，不是 HMD 全字段详情。
- 后续若页面需要 HMD 长字段，再扩展字段，不复用 publish service。
