# POST /api/v1/decentralized_community/detail

## 1. 入口

- 鉴权：需要 `publish + landlord` session。
- handler：`internal/handler/v1/publish/community.(*Handler).Detail`
- service：`internal/service/publish.(*decentralizedCommunityService).GetDecentralizedCommunity`

## 2. 处理逻辑

1. handler 绑定 `id`。
2. service 构造 `PublishScope`。
3. 先读 `hs_hmd_decentralized`。
4. 用 `community_id` 去 root scope relation 判断 owner。
5. 权限通过后返回，无权访问按 not found 处理。

## 3. 数据读写

读取：

- `hs_hmd_decentralized`
- `hs_hpd_root_scope_relation`

写入：

- 无

## 4. 失败返回

- `id` 非法：`InvalidParam`
- 小区不存在：`NotFound`
- 当前房东无权访问：`NotFound`
- 查询失败：`DatabaseError`
