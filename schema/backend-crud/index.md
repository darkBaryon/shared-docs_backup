# Backend CRUD

本文档记录当前 Go 后端已经落地的 repository CRUD / query 方法，作为数据库设计文档的实现补充。

## 1. 账号与认证

### `hs_usr_user`

- `Create(user *model.User) error`
  - 用途：创建用户主档。
  - 参数：`user` 必须包含 `phone`。
  - 返回：创建失败返回 error。
- `TouchLastActive(userID bson.ObjectID, lastActiveAt int64) error`
  - 用途：更新用户最近活跃时间。
  - 参数：`userID` 用户 ID；`lastActiveAt` 秒级时间戳。
  - 返回：更新失败返回 error。
- `FindByPhone(phone string) (*model.User, error)`
  - 用途：按手机号查询有效用户。
  - 参数：`phone` 手机号。
  - 返回：命中返回用户；未命中返回 `nil, nil`。

### `hs_usr_auth`

- `Create(auth *model.UserAuth) error`
  - 用途：创建微信认证绑定记录。
  - 参数：`auth` 必须包含 `user_id`、`auth_provider`、`openid`。
  - 返回：创建失败返回 error。
- `TouchLastLogin(authID bson.ObjectID, lastLoginAt int64, lastLoginIP string) error`
  - 用途：更新最近登录时间与 IP。
  - 参数：`authID` 认证记录 ID；`lastLoginAt` 秒级时间戳；`lastLoginIP` 登录 IP。
  - 返回：更新失败返回 error。
- `FindByOpenID(authProvider, openID string) (*model.UserAuth, error)`
  - 用途：按认证来源和 openid 查询绑定关系。
  - 参数：`authProvider` 认证来源；`openID` 微信 openid。
  - 返回：命中返回认证记录；未命中返回 `nil, nil`。
- `FindByUnionID(authProvider, unionID string) (*model.UserAuth, error)`
  - 用途：按认证来源和 unionid 查询绑定关系。
  - 参数：`authProvider` 认证来源；`unionID` 微信 unionid。
  - 返回：命中返回认证记录；未命中返回 `nil, nil`。
- `FindByUserID(userID bson.ObjectID) (*model.UserAuth, error)`
  - 用途：按用户 ID 查询认证绑定。
  - 参数：`userID` 用户 ID。
  - 返回：命中返回认证记录；未命中返回 `nil, nil`。

### `hs_usr_profile_ext`

- `Create(profile *model.UserProfileExt) error`
  - 用途：创建用户偏好扩展信息。
  - 参数：`profile` 必须包含 `user_id`。
  - 返回：创建失败返回 error。
- `UpsertByUserID(userID bson.ObjectID, fields bson.M) (matched bool, err error)`
  - 用途：按用户 ID 局部更新偏好；不存在则插入。
  - 参数：`userID` 用户 ID；`fields` 需要写入的字段集合。
  - 返回：`matched=true` 表示更新已有记录；`matched=false` 表示新插入。
- `FindByUserID(userID bson.ObjectID) (*model.UserProfileExt, error)`
  - 用途：按用户 ID 查询偏好扩展信息。
  - 参数：`userID` 用户 ID。
  - 返回：命中返回偏好记录；未命中返回 `nil, nil`。

## 2. 房源主数据（HMD）

### `hs_hmd_centralized`

- `Create(entity *model.HmdCentralized) error`
  - 用途：创建集中式项目主档。
  - 参数：`entity` 必须包含 `project_name`、`city`。
  - 返回：创建失败返回 error。
- `FindByID(id bson.ObjectID) (*model.HmdCentralized, error)`
  - 用途：按项目 ID 查询主档。
  - 参数：`id` 项目 ID。
  - 返回：命中返回项目；未命中返回 `nil, nil`。
- `ListByCity(city string) ([]model.HmdCentralized, error)`
  - 用途：按城市列出有效集中式项目。
  - 参数：`city` 城市名。
  - 返回：项目列表。
- `ListByCityAndDistrict(city, district string) ([]model.HmdCentralized, error)`
  - 用途：按城市和区域列出有效集中式项目。
  - 参数：`city` 城市名；`district` 区域名。
  - 返回：项目列表。
- `UpdateBaseInfo(id bson.ObjectID, fields bson.M) error`
  - 用途：更新项目基础信息。
  - 参数：`id` 项目 ID；`fields` 需要更新的字段集合。
  - 返回：更新失败返回 error。

### `hs_hmd_building`

- `Create(entity *model.HmdBuilding) error`
  - 用途：创建楼栋主档。
  - 参数：`entity` 必须包含 `project_id`、`building_name`。
  - 返回：创建失败返回 error。
- `FindByID(id bson.ObjectID) (*model.HmdBuilding, error)`
  - 用途：按楼栋 ID 查询主档。
  - 参数：`id` 楼栋 ID。
  - 返回：命中返回楼栋；未命中返回 `nil, nil`。
- `FindByProjectAndName(projectID bson.ObjectID, buildingName string) (*model.HmdBuilding, error)`
  - 用途：按项目和楼栋名称查询主档。
  - 参数：`projectID` 项目 ID；`buildingName` 楼栋名称。
  - 返回：命中返回楼栋；未命中返回 `nil, nil`。
- `ListByProjectID(projectID bson.ObjectID) ([]model.HmdBuilding, error)`
  - 用途：列出某个项目下的楼栋。
  - 参数：`projectID` 项目 ID。
  - 返回：楼栋列表。
- `UpdateBaseInfo(id bson.ObjectID, fields bson.M) error`
  - 用途：更新楼栋基础信息。
  - 参数：`id` 楼栋 ID；`fields` 需要更新的字段集合。
  - 返回：更新失败返回 error。

### `hs_hmd_decentralized`

- `Create(entity *model.HmdDecentralized) error`
  - 用途：创建分散式小区主档。
  - 参数：`entity` 必须包含 `community_name`、`city`。
  - 返回：创建失败返回 error。
- `FindByID(id bson.ObjectID) (*model.HmdDecentralized, error)`
  - 用途：按分散式主档 ID 查询。
  - 参数：`id` 主档 ID。
  - 返回：命中返回记录；未命中返回 `nil, nil`。
- `FindByCommunity(city, district, communityName string) (*model.HmdDecentralized, error)`
  - 用途：按城市、区域、小区名查询分散式主档。
  - 参数：`city` 城市名；`district` 区域名；`communityName` 小区名。
  - 返回：命中返回记录；未命中返回 `nil, nil`。
- `ListByCity(city string) ([]model.HmdDecentralized, error)`
  - 用途：按城市列出分散式主档。
  - 参数：`city` 城市名。
  - 返回：主档列表。
- `ListByDistrict(city, district string) ([]model.HmdDecentralized, error)`
  - 用途：按城市和区域列出分散式主档。
  - 参数：`city` 城市名；`district` 区域名。
  - 返回：主档列表。
- `UpdateBaseInfo(id bson.ObjectID, fields bson.M) error`
  - 用途：更新分散式主档基础信息。
  - 参数：`id` 主档 ID；`fields` 需要更新的字段集合。
  - 返回：更新失败返回 error。

### `hs_hmd_room_type_centralized`

- `Create(entity *model.HmdRoomTypeCentralized) error`
  - 用途：创建集中式房型模板。
  - 参数：`entity` 必须包含 `room_type_name`。
  - 返回：创建失败返回 error。
- `FindByID(id bson.ObjectID) (*model.HmdRoomTypeCentralized, error)`
  - 用途：按房型模板 ID 查询。
  - 参数：`id` 房型模板 ID。
  - 返回：命中返回记录；未命中返回 `nil, nil`。
- `FindByProjectAndName(projectID bson.ObjectID, roomTypeName string) (*model.HmdRoomTypeCentralized, error)`
  - 用途：按项目和房型名称查询模板。
  - 参数：`projectID` 项目 ID；`roomTypeName` 房型名称。
  - 返回：命中返回记录；未命中返回 `nil, nil`。
- `ListByProjectID(projectID bson.ObjectID) ([]model.HmdRoomTypeCentralized, error)`
  - 用途：列出项目下的房型模板。
  - 参数：`projectID` 项目 ID。
  - 返回：房型模板列表。
- `ListByBuildingID(buildingID bson.ObjectID) ([]model.HmdRoomTypeCentralized, error)`
  - 用途：列出楼栋下的房型模板。
  - 参数：`buildingID` 楼栋 ID。
  - 返回：房型模板列表。
- `UpdateBaseInfo(id bson.ObjectID, fields bson.M) error`
  - 用途：更新房型模板基础信息。
  - 参数：`id` 房型模板 ID；`fields` 需要更新的字段集合。
  - 返回：更新失败返回 error。

### `hs_hmd_room_centralized`

- `Create(entity *model.HmdRoomCentralized) error`
  - 用途：创建集中式房间实例。
  - 参数：`entity` 必须包含 `room_no`、`rent_mode`。
  - 返回：创建失败返回 error。
- `FindByID(id bson.ObjectID) (*model.HmdRoomCentralized, error)`
  - 用途：按房间 ID 查询。
  - 参数：`id` 房间 ID。
  - 返回：命中返回记录；未命中返回 `nil, nil`。
- `FindByBuildingAndRoomNo(buildingID bson.ObjectID, roomNo string) (*model.HmdRoomCentralized, error)`
  - 用途：按楼栋和房号查询房间。
  - 参数：`buildingID` 楼栋 ID；`roomNo` 房号。
  - 返回：命中返回记录；未命中返回 `nil, nil`。
- `ListByProjectID(projectID bson.ObjectID) ([]model.HmdRoomCentralized, error)`
  - 用途：列出项目下的房间。
  - 参数：`projectID` 项目 ID。
  - 返回：房间列表。
- `ListByBuildingID(buildingID bson.ObjectID) ([]model.HmdRoomCentralized, error)`
  - 用途：列出楼栋下的房间。
  - 参数：`buildingID` 楼栋 ID。
  - 返回：房间列表。
- `UpdateBaseInfo(id bson.ObjectID, fields bson.M) error`
  - 用途：更新房间基础信息。
  - 参数：`id` 房间 ID；`fields` 需要更新的字段集合。
  - 返回：更新失败返回 error。
- `UpdateStatus(id bson.ObjectID, roomStatus int) error`
  - 用途：更新房间状态。
  - 参数：`id` 房间 ID；`roomStatus` 房间状态值。
  - 返回：更新失败返回 error。

### `hs_hmd_room_decentralized`

- `Create(entity *model.HmdRoomDecentralized) error`
  - 用途：创建分散式房间实例。
  - 参数：`entity` 必须包含 `decentralized_id`、`room_no`、`rent_mode`。
  - 返回：创建失败返回 error。
- `FindByID(id bson.ObjectID) (*model.HmdRoomDecentralized, error)`
  - 用途：按房间 ID 查询。
  - 参数：`id` 房间 ID。
  - 返回：命中返回记录；未命中返回 `nil, nil`。
- `FindByDecentralizedAndRoomNo(decentralizedID bson.ObjectID, roomNo string) (*model.HmdRoomDecentralized, error)`
  - 用途：按分散式主档和房号查询房间。
  - 参数：`decentralizedID` 分散式主档 ID；`roomNo` 房号。
  - 返回：命中返回记录；未命中返回 `nil, nil`。
- `ListByDecentralizedID(decentralizedID bson.ObjectID) ([]model.HmdRoomDecentralized, error)`
  - 用途：列出某个小区主档下的房间。
  - 参数：`decentralizedID` 分散式主档 ID。
  - 返回：房间列表。
- `UpdateBaseInfo(id bson.ObjectID, fields bson.M) error`
  - 用途：更新房间基础信息。
  - 参数：`id` 房间 ID；`fields` 需要更新的字段集合。
  - 返回：更新失败返回 error。
- `UpdateStatus(id bson.ObjectID, roomStatus int) error`
  - 用途：更新房间状态。
  - 参数：`id` 房间 ID；`roomStatus` 房间状态值。
  - 返回：更新失败返回 error。

## 3. 发布与展示（HPD）

### `hs_hpd_listing`

- `Create(entity *model.HpdListing) error`
  - 用途：创建发布主实体。
  - 参数：`entity` 必须包含 `source_type`、`source_id`、`asset_mode`、`listing_status`。
  - 返回：创建失败返回 error。
- `FindByID(id bson.ObjectID) (*model.HpdListing, error)`
  - 用途：按 listing ID 查询发布主实体。
  - 参数：`id` listing ID。
  - 返回：命中返回记录；未命中返回 `nil, nil`。
- `FindBySource(sourceType model.HpdSourceType, sourceID bson.ObjectID) (*model.HpdListing, error)`
  - 用途：按 HMD 来源对象查询发布主实体。
  - 参数：`sourceType` 来源类型；`sourceID` 来源对象 ID。
  - 返回：命中返回记录；未命中返回 `nil, nil`。
- `UpsertBySource(entity *model.HpdListing) (*model.HpdListing, error)`
  - 用途：按 `source_type + source_id` 创建或更新发布主实体。
  - 参数：`entity` 发布主实体。
  - 返回：写入后的发布主实体。
- `UpdateLifecycleFields(id bson.ObjectID, fields bson.M) error`
  - 用途：更新发布生命周期字段。
  - 参数：`id` listing ID；`fields` 仅允许 `listing_status`、`published_at`、`offline_at`。
  - 返回：更新失败返回 error。
- `UpdateStatus(id bson.ObjectID, listingStatus model.HpdListingStatus) error`
  - 用途：更新发布状态。
  - 参数：`id` listing ID；`listingStatus` 发布状态。
  - 返回：更新失败返回 error。

### `hs_hpd_miniapp_listing`

- `Create(entity *model.HpdMiniappListing) error`
  - 用途：创建小程序展示 read model。
  - 参数：`entity` 必须包含 `listing_id`、`source_type`、`source_id`、`asset_mode`、`rent_mode`、`city`、`title`。
  - 返回：创建失败返回 error。
- `FindByID(id bson.ObjectID) (*model.HpdMiniappListing, error)`
  - 用途：按 `_id` 查询小程序展示记录。
  - 参数：`id` 文档 ID。
  - 返回：命中返回记录；未命中返回 `nil, nil`。
- `FindByListingID(listingID bson.ObjectID) (*model.HpdMiniappListing, error)`
  - 用途：按统一 listing ID 查询小程序展示记录。
  - 参数：`listingID` 关联 `hs_hpd_listing._id`。
  - 返回：命中返回记录；未命中返回 `nil, nil`。
- `FindBySource(sourceType model.HpdSourceType, sourceID bson.ObjectID) (*model.HpdMiniappListing, error)`
  - 用途：按 HMD 来源对象查询小程序展示记录。
  - 参数：`sourceType` 来源类型；`sourceID` 来源对象 ID。
  - 返回：命中返回记录；未命中返回 `nil, nil`。
- `FindOnlineDetail(listingID bson.ObjectID) (*model.HpdMiniappListing, error)`
  - 用途：按 listing ID 查询小程序在线详情。
  - 参数：`listingID` 关联 `hs_hpd_listing._id`。
  - 返回：在线命中返回记录；未命中或非在线返回 `nil, nil`。
- `UpsertByListingID(entity *model.HpdMiniappListing) (*model.HpdMiniappListing, error)`
  - 用途：按 `listing_id` 创建或更新小程序展示记录。
  - 参数：`entity` 小程序展示记录。
  - 返回：写入后的小程序展示记录。
- `UpdateProjectionFields(listingID bson.ObjectID, fields bson.M) error`
  - 用途：按 `listing_id` 局部更新小程序展示字段。
  - 参数：`listingID` 关联 `hs_hpd_listing._id`；`fields` 只允许展示字段，不允许改 `listing_id/source_id` 等身份字段。
  - 返回：更新失败返回 error。
- `SearchMiniapp(search MiniappListingSearchFilter) ([]model.HpdMiniappListing, error)`
  - 用途：查询小程序在线房源列表。
  - 参数：`search` 支持城市、区域、商圈、租住方式、价格区间、分页。
  - 返回：按 `weight_score`、`updated_at` 倒序的在线房源列表。

## 4. 说明

- 当前文档只记录已经在 Go 后端仓库中落地的方法。
- 若后续新增 `hac`、`lead` 等模块 repository，需要同步追加到本文件。
- 若数据库设计调整，应同时更新 `schema/db-design/` 与本文件。
