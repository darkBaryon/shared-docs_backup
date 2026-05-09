# System Map

## 前端

- 小程序：C 端用户找房入口。
- 发房系统：房源录入、发布、房东侧协作入口。
- 后台管理：运营、审核、数据维护入口。

## 后端

- Go 后端 `house-manager`：
  - auth
  - 用户主档
  - HMD 主数据
  - publish 发房业务
  - 后续 HPD、后台管理能力
- Python 后端 `ai_house`：
  - AI chat 相关能力
  - 迁移业务行为参考

## 数据层

- MongoDB：主业务数据。
- Redis：session、缓存、临时状态。

## 关系

```text
小程序
发房系统
后台管理
  -> Go 后端 house-manager
    -> MongoDB / Redis

小程序 AI 对话
  -> Python AI chat
    -> 视业务需要读取 Go 后端或 MongoDB 中的房源与用户数据
```

Go 迁移目标是重建清晰的 Go 分层架构，不继承 Python 项目的历史技术边界。
