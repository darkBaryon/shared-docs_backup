# POST /api/v1/decentralized_community/list

## 1. 入口

- 鉴权：需要 `publish + landlord` session。
- handler：`internal/handler/v1/publish/community.(*Handler).List`
- service：`internal/service/publish.(*decentralizedCommunityService).ListDecentralizedCommunities`

## 2. 当前查询主线

这条接口已经改成：

1. 先从 `hs_hpd_root_scope_relation` 查当前房东可见的 `community_ids`
2. 再带着这些 ids 去查 `hs_hmd_decentralized`
3. 同时叠加 `city / district` 业务筛选

## 3. 数据读写

读取：

- `hs_hpd_root_scope_relation`
- `hs_hmd_decentralized`

写入：

- 无

## 4. 返回逻辑

- 当前房东无任何 root scope：直接返回空列表
- 当前房东有 root scope，但业务筛选后无结果：返回空列表

## 5. 失败返回

- scope 查询失败：`DatabaseError`
- HMD 查询失败：`DatabaseError`

## 6. 关键点

- owner scope 是第一收口条件
- city/district 只是业务筛选条件
- `city` 不是必填；不传时表示返回当前房东全部可见小区
