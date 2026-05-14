# 管理端 API 实施计划

## 背景

后台管理端尚未开始实现。管理端需要独立设计端侧 HTTP 表达层和端侧应用服务。

## 目标

建立管理端后端入口：

```text
handler/v1/admin/*
service/admin/*
```

管理端可以复用内部 `domain/*` 和 `repository/*` 能力，但不能复用小程序或 publish 的端侧应用 service。

## 约束

- 不做旧接口、旧字段、旧数据库兼容。
- 新增接口路径必须遵守 `POST /api/v1/{module}/{action}`。
- 管理端鉴权、权限和操作边界独立设计，不挂在 miniapp auth 或 publish service 上。
- 管理端 API 契约先写入 `api/`，再实现代码。

## 下一步

1. 明确管理端第一批业务范围。
2. 在 `api/` 新增或更新管理端 API 契约。
3. 在 `backend/` 补充管理端服务边界。
4. 实现 `handler/v1/admin/*` 和 `service/admin/*`。
5. 更新 [../status.md](../status.md)。
