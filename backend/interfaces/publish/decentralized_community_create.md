# POST /api/v1/decentralized_community/create

## 1. 入口

- 鉴权：需要 `publish + landlord` session。
- handler：`internal/handler/v1/publish/community.(*Handler).Create`
- service：`internal/service/publish.(*decentralizedCommunityService).CreateDecentralizedCommunity`

## 2. 处理逻辑

1. handler 绑定小区基础字段。
2. service 从 context 读取 publish principal。
3. 调 `domain/hmd.CreateDecentralizedCommunity` 写入 `hs_hmd_decentralized`。
4. HMD 返回 `HmdMutationResult`。
5. service 调 `listingprojection.Apply(changes)` 刷新 HPD projection。
6. 调 `publishaccess.UpsertRootScopeForPrincipal` 写入 `hs_hpd_root_scope_relation`：
   - `root_type=decentralized_community`
   - `root_id=community_id`
   - `owner_landlord_id=current landlord`
7. 任一步失败时回滚刚创建的 HMD 小区。
8. 返回创建后的 community。

## 3. 数据读写

写入：

- `hs_hmd_decentralized`
- 受 change 影响的 HPD projection
- `hs_hpd_root_scope_relation`

## 4. 失败返回

- 入参不合法：`InvalidParam`
- HMD 写入失败：`DatabaseError`
- HPD projection 失败：内部错误 / projection error
- root scope 写入失败：内部错误 / database error

## 5. 关键约束

- 空小区创建后，当前房东必须立刻可见
