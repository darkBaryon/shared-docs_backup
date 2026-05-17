# POST /api/v1/house_building/list

## 1. 当前状态

该接口用于后台房源管理第二层级的楼栋列表。

目标：

- 后台员工在选定项目后查看楼栋列表。
- 分散式小区没有楼栋时，前端可直接跳过该层级进入房间列表。

## 2. 入口

- 鉴权：需要 Bearer token。
- 权限：需要 `house.view`。
- handler：`internal/handler/v1/admin/house.(*Handler).ListBuildings`
- service：`internal/service/admin/house.(*Service).ListBuildings`
- 主读集合：`hs_hpd_admin_listing`

## 3. 请求体

```json
{
  "root_id": "root_object_id",
  "page": 1,
  "page_size": 20
}
```

## 4. 失败返回

| 场景 | 错误码 | 对外中文提示 |
| --- | --- | --- |
| 缺少 `house.view` 权限 | `Forbidden` | `当前账号无权查看楼栋列表` |
| 请求体格式错误 | `InvalidParam` | `请求参数格式不正确` |
| `root_id` 缺失或格式错误 | `InvalidParam` | `项目/小区参数不正确` |
| 查询失败 | `DatabaseError` | `获取楼栋列表失败，请稍后重试` |

## 5. 关键约束

- 只返回集中式项目下存在房间的楼栋。
- 分散式小区查询结果允许为空列表，这是正常情况。
