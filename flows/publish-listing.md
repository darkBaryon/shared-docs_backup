# Publish Listing Flow

## 当前第一期

```text
发房系统前端
  -> POST /api/v1/publish/{action}
    -> PublishHandler
      -> PublishService
        -> domain/hmd.Service 写 HMD
          -> repository/hmd
            -> MongoDB HMD collections
        -> domain/hpd.Service.Apply(changes)
```

当前 `domain/hpd.Service.Apply` 已接入第一期小程序 HPD projector，HMD 房间及父级主数据变更会刷新对应展示快照。

当前第一期后端链路已通过真实服务联调：

- 受保护 publish 路由通过 Redis Session 鉴权。
- HMD 六个 collection 可以真实落库。
- 集中式项目、楼栋、房型、集中式房间、分散式小区、分散式房间主链路已验证。
- 分散式房间当前不写 `room_type_id`。

发房前端可以基于这条链路开始 HMD 录入与维护的 E2E。

## 当前 HPD projector

```text
PublishService
  -> domain/hmd.Service 写主数据，返回 HmdMutationResult
  -> domain/hpd.Service.Apply(changes)
    -> MiniappProjector
      -> 重新读取 HMD 当前源数据
      -> map HMD to HPD
      -> upsert HPD
```

## 后续 outbox 接入

如果同步投影无法满足可靠性要求，可以升级为：

```text
HMD 写入事务
  -> 写 HMD
  -> 写 outbox event

worker
  -> 读取 outbox event
  -> domain/hpd.Service.Apply(changes)
```

Projector 仍然复用 `Apply(changes)` 后面的投影逻辑。
