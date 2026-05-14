# 前端目标架构迁移计划

## 背景

当前代码仍是旧结构：

```text
pages/*
  -> services/{module}
    -> api/{module}.ts
      -> utils/http / utils/request
```

目标结构以 [../../../frontend/development-spec.md](../../../frontend/development-spec.md) 和 [../../../frontend/miniapp.md](../../../frontend/miniapp.md) 为准：

```text
pages/*
components/{module}
assets/css/{module}
api/{module}.ts
models/{model}.ts
utils/*
```

## 迁移重点

1. `api/*` 从 URL 常量升级为完整 API 函数。
2. `services/house/adapters.ts`、`services/favorite/mapper.ts`、`services/history/mapper.ts` 合并为统一 `models/listing.ts`。
3. `discover` 搜索面板 UI 状态回收到组件内，页面只接收 `search` 事件和查询参数。
4. 页面组件从 `pages/*` 迁到 `components/{module}`，样式迁到 `assets/css/{module}`。
5. `utils` 不再依赖 `pages` 常量，storage key 移到 `utils/storageKeys.ts` 或 `constants`。

## 建议顺序

1. `api/house.ts`：补 `searchHouses`、`getPublicHouseDetail` API 函数和 request/response 类型。
2. `models/listing.ts`：抽统一 HPD listing card mapper。
3. `api/favorite.ts`、`api/history.ts`：补 API 函数和 request/response 类型。
4. `favorite/history` 页面改为复用 `models/listing.ts`。
5. `discover` 搜索面板状态回收到组件内。
6. `components/discover`：迁移 discover 私有组件和常量，样式放入 `assets/css/discover`。
7. `utils/storageKeys.ts`：统一 token/profile/favorite/history cache key。

## 迁移原则

- 不一次性大重构。
- 每次只迁一个 feature 或一个 API 模块。
- 迁移后必须保持真机 E2E 可跑。

## 完成后更新

- [../status.md](../status.md)
- [../../../frontend/miniapp.md](../../../frontend/miniapp.md)
- [../../../frontend/development-spec.md](../../../frontend/development-spec.md)
