# 小程序前端工作流状态

本文只保留小程序前端当前状态和任务入口。接口、字段和 ID 口径以 [../../api/miniapp-api.md](../../api/miniapp-api.md) 为准。

## 当前事实

小程序前端 API 调用当前分为四层：

```text
pages/*
  -> services/{module}
    -> api/{module}.ts
      -> utils/http / utils/request
        -> POST /api/v1/{module}/{action}
```

当前重要约束：

- 小程序房源统一使用 `listing_id`，禁止继续使用旧 `house_id` 作为主链路字段。
- Miniapp API 请求和响应字段统一使用 `snake_case`。
- 前端只消费 miniapp API DTO，不直接消费 Go model JSON 或旧 Python 返回结构。
- HPD 房源展示只使用 `MiniappListingItem` / `MiniappListingDetail` 字段。

## 已完成

- 请求层：按 `code === 0` 判断成功，失败读取 Go 后端 `error` 字段，Bearer token 透传。
- Auth：`wechat_login`、`wechat_register`。
- HPD 房源公开读链路：`house/search`、`house/public_detail`。
- `feature_flags`：第一期只提交 API 契约允许值。
- 收藏接口：`favorite/add`、`favorite/remove`、`favorite/list`。
- 足迹接口：`history/add`、`history/list`。
- 用户接口类型：`user/profile`、`user/update_profile`、`user/dashboard`。
- 旧 house 管理接口清理：删除旧管理接口常量和 service。

## 当前任务

- [前端目标架构迁移](./active/frontend-architecture.md)
- [真机 E2E 验证](./active/device-e2e-validation.md)

## 范围说明

`chat/send` 本期继续使用旧接口，不纳入本轮 HPD miniapp API 字段对齐范围。

允许旧字段继续出现在：

- `src/services/chat`
- `src/pages/ai`

## 必读文档

1. [../../api/miniapp-api.md](../../api/miniapp-api.md)：小程序 API 契约。
2. [../backend-go/status.md](../backend-go/status.md)：Go 后端当前进度。
3. [../../backend/miniapp-hpd.md](../../backend/miniapp-hpd.md)：小程序 HPD 展示层。
4. [../../frontend/miniapp.md](../../frontend/miniapp.md)：小程序前端约定。
5. [../../schema/db-design/v4/index.md](../../schema/db-design/v4/index.md)：数据库设计入口。

## 维护规则

- 新增或完成小程序任务时，更新本文的“已完成”和“当前任务”。
- 具体迁移计划写入 `active/`，不要塞回本文。
- review report 写入 `reviews/YYYY-MM-DD-topic.md`。
- 长期有效的 API 口径直接写入 [../../api/miniapp-api.md](../../api/miniapp-api.md)。
