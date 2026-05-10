# Miniapp Search Flow

当前小程序找房读接口已按 HPD read model 接入。

下一步先实现小程序 HPD，不先做后台管理 HPD 或房东端 HPD。

## 目标形态

```text
小程序
  -> Go 后端房源查询 API
    -> handler/v1/miniapp/house
      -> service/miniapp/house
        -> repository/hpd.MiniappListingRepository
          -> hs_hpd_miniapp_listing
```

HPD 未完成前，不应让小程序直接依赖 HMD 主数据结构。

边界约束：

- `house` 是对外 API 模块；`miniapp` 是端类型，用于代码目录分组，不进入 URL。
- `service/miniapp/house` 只读 HPD read model，不读 HMD。
- `handler/v1/miniapp/house` 不直接调用 `repository/hpd`，也不调用 `service/hpd` projector。
- `service/hpd` 继续只负责 HPD projection / lifecycle / `Apply(changes)`。

## 写入来源

```text
发房端 publish API
  -> PublishService
    -> hmd.Service 写 HMD
    -> hpd.Service.Apply(changes)
      -> MiniappProjector
        -> 生成 hs_hpd_listing
        -> 生成 hs_hpd_miniapp_listing
```

第一期发布源：

- `hs_hmd_room_centralized`
- `hs_hmd_room_decentralized`

第一期不把集中式房型模板作为小程序发布源。

## 接口规划

小程序第一期查询接口：

- `POST /api/v1/house/search`
- `POST /api/v1/house/public_detail`

路径命名遵守 `POST /api/v{version}/{模块}/{动作}`。找房业务归属 `house` 模块，不使用 `/api/v1/miniapp/listing/search` 这类多级业务路径。

详细规划见 [../backend/miniapp-hpd.md](../backend/miniapp-hpd.md)。
