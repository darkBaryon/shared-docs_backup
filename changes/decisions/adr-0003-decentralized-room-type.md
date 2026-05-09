# ADR 0003: 分散式房间暂不提供房型模型

## 决策

分散式房间当前不提供独立房型模型，也不引用集中式房型模型。

## 约束

- `hs_hmd_room_decentralized` 当前不包含 `room_type_id`。
- 分散式房间 API 当前不接收 `room_type_id`。
- 不允许复用 `hs_hmd_room_type_centralized` 作为分散式房型。

## 后续

如果后续业务需要分散式房型，必须单独设计分散式房型模型和集合。
