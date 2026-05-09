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
