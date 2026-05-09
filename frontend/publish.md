# Frontend Publish

发房系统前端以 [../api/publish.md](../api/publish.md) 为接口依据。

## 当前后端状态

- 发房端后端第一期 HMD 链路已通过真实服务联调。
- 前端可以开始接入 HMD 录入、详情、列表、更新和房态流转。
- 当前接口已经走 Redis Session 鉴权，联调时需要 `Authorization: Bearer <token>`。
- HPD 小程序 projector 已接入，暂时仍不要把 HMD 结果当成小程序展示层 API 数据。

## 约定

- API 路径使用 `POST /api/v1/publish/{action}`。
- 表单枚举值以 schema 为准。
- 分散式房间当前不提供房型选择。
- 发房系统和后台管理系统不是同一个前端。
