# ADR 0002: Publish / HMD / HPD 边界

## 决策

`publish` 是发房业务域，不是 HMD CRUD 直通层。

当前结构：

```text
PublishService
  -> publish/hmd.Service
    -> HmdMutationResult{Entity, Changes}
  -> publish/hpd.Service.Apply(changes)
```

## 说明

- HMD 负责房源主数据。
- HPD 负责发布与展示层数据。
- 发房系统第一期先落 HMD 能力。
- HMD 实现位于 `internal/service/publish/hmd`。
- HPD 实现位于 `internal/service/publish/hpd`。
- `internal/service/publish` 顶层只保留 facade，不承载 HMD 具体实现。
- `publish/hmd.Service` 不直接调用 HPD。
- HMD 写操作返回 changes，PublishService 统一派发给 `hpd.Service`。
- HPD 当前 `Apply` 是 no-op 预留点，后续接入 projector 或 outbox worker 复用的投影逻辑。
- 错误语义由 service 层返回 `errcode`，handler 不通过字符串归类错误。

## 后续

- projector 负责根据 changes 重新读取 HMD 源数据并生成 HPD。
- outbox 只负责可靠投递 changes，不改变 projector 的投影逻辑。
