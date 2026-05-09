# Publish Listing Flow

## 当前第一期

```text
发房系统前端
  -> POST /api/v1/publish/{action}
    -> PublishHandler
      -> PublishService
        -> publish/hmd.Service 写 HMD
          -> repository/hmd
            -> MongoDB HMD collections
        -> publish/hpd.Service.Apply(changes)
```

当前 `publish/hpd.Service.Apply` 是 no-op，第一期只保证 handler 和 PublishService 已支持 change 派发架构。

当前第一期后端链路已通过真实服务联调：

- 受保护 publish 路由通过 Redis Session 鉴权。
- HMD 六个 collection 可以真实落库。
- 集中式项目、楼栋、房型、集中式房间、分散式小区、分散式房间主链路已验证。
- 分散式房间当前不写 `room_type_id`。

发房前端可以基于这条链路开始 HMD 录入与维护的 E2E。

## 后续 HPD projector 接入

```text
PublishService
  -> publish/hmd.Service 写主数据，返回 HmdMutationResult
  -> publish/hpd.Service.Apply(changes)
    -> HpdProjector
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
  -> publish/hpd.Service.Apply(changes)
```

Projector 仍然复用 `Apply(changes)` 后面的投影逻辑。
