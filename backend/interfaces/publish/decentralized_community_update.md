# POST /api/v1/decentralized_community/update

## 1. 入口

- 鉴权：需要 `publish + landlord` session。
- handler：`internal/handler/v1/publish/community.(*Handler).Update`
- service：`internal/service/publish.(*decentralizedCommunityService).UpdateDecentralizedCommunity`

## 2. 处理逻辑

1. handler 绑定小区更新字段。
2. service 构造 `PublishScope`。
3. 校验当前房东是否拥有该 root。
4. 通过后调用 `domain/hmd.UpdateDecentralizedCommunity` 更新 `hs_hmd_decentralized`。
5. HMD 返回 change。
6. service 调 `listingprojection.Apply(changes)` 刷新相关 HPD projection。
7. 返回更新后的 community。

## 3. 数据读写

读取：

- `hs_hpd_root_scope_relation`
- `hs_hmd_decentralized`

写入：

- `hs_hmd_decentralized`
- 相关 HPD projection

## 4. 失败返回

- `id` 非法：`InvalidParam`
- 当前房东无权更新：`NotFound`
- 小区不存在：`NotFound`
- HMD 更新失败：`DatabaseError`
- projection 刷新失败：内部错误 / projection error
