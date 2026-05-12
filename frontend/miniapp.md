# Frontend Miniapp

小程序前端以 [../api/miniapp-api.md](../api/miniapp-api.md) 为接口依据。

## 1. 基本约定

- 登录态使用 Bearer token。
- 页面字段展示以 API 和 schema 为准。
- 找房列表和详情后续读取 HPD 展示层数据，不直接依赖 HMD。
- 小程序 HPD 后端规划见 [../backend/miniapp-hpd.md](../backend/miniapp-hpd.md)。

## 2. 目标目录结构

小程序前端采用页面驱动的 feature 结构：

```text
src/
  api/
    auth.ts
    house.ts
    favorite.ts
    history.ts
    user.ts
    chat.ts

  models/
    listing.ts
    profile.ts

  pages/
    discover/
      discover.vue
    house-detail/
      house-detail.vue
    favorites/
      favorites.vue
    history/
      history.vue
    login/
      login.vue
    profile/
      profile.vue

  components/
    discover/
      DiscoverSearchPanel.vue
      DiscoverHouseList.vue
      constants.ts

    house-detail/
      HouseDetailGallery.vue
      HouseDetailInfo.vue

  assets/
    css/
      discover/
        discover.scss
        DiscoverSearchPanel.scss
      house-detail/
        house-detail.scss

  styles/
    app.scss

  utils/
    http.ts
    request.ts
    image.ts
    auth.ts
    storageKeys.ts
```

说明：

- `pages/` 只表达小程序路由入口。
- 页面组件统一放入 `src/components/{module}/`，按业务模块分目录。
- 页面样式和组件样式统一放入 `src/assets/css/{module}/`。
- API 定义不放到 `components/` 或 `pages/`，统一在 `src/api/`。

## 3. API 层

`src/api/` 负责完整接口定义，不只放 URL。

每个 API 文件应包含：

```text
URL
method
request type
response data type
调用函数
```

以找房搜索为例：

```ts
export interface SearchHouseRequest {
  city?: string;
  district?: string;
  biz_area?: string;
  rent_mode?: "whole" | "shared";
  asset_mode?: "centralized" | "decentralized";
  min_price?: number;
  max_price?: number;
  keyword?: string;
  feature_flags?: Array<"subway" | "elevator" | "pet_friendly" | "cooking_allowed">;
  page?: number;
  page_size?: number;
}

export interface SearchHouseResponse {
  list: MiniappListingItem[];
  page: number;
  page_size: number;
  total: number;
}

export function searchHouses(data: SearchHouseRequest) {
  return http.post<SearchHouseResponse>("/house/search", data);
}
```

禁止：

- `api/` 只导出 URL 常量。
- 把请求方式、payload 类型、response 类型散到 `services/`。
- 页面直接调用 `http.post("/house/search", ...)`。

## 4. Model / Mapper 层

`src/models/` 负责后端 DTO 到 UI ViewModel 的转换。

HPD 房源卡片必须统一映射：

```ts
export function toListingCard(item: MiniappListingItem): ListingCardVM {
  return {
    id: item.listing_id,
    title: item.title || item.community_name || "房源信息",
    district: [item.district, item.biz_area || item.community_name].filter(Boolean).join(" · "),
    priceText: item.price_text || (item.price ? `${item.price}元/月` : "价格待补充"),
    tags: item.platform_tags?.slice(0, 4) || item.listing_facilities?.slice(0, 4) || [],
    cover: normalizeImageList(item.images)[0] || "",
  };
}
```

找房列表、收藏列表、足迹列表都复用这套 listing mapper。足迹只额外追加 `sourceText`、`viewedAtText` 等页面展示字段。

禁止：

- favorite/history/house 分别复制一份 HPD 卡片映射。
- 在 mapper 中发请求、跳页面、弹 toast。

## 5. Page 与 Component

### 5.1 Page 职责

页面负责：

- 页面级加载流程。
- 调用 API。
- 调用 model/mapper。
- 页面跳转。
- 组合 feature components。

页面不负责：

- 组件内部表单状态。
- 大量 props/emits 中转。
- 直接拼接口 URL。

### 5.2 Component 职责

组件负责：

- 自己的 UI 状态。
- 与 UI 强绑定的交互，例如搜索面板展开、筛选项选择、输入框值、重置筛选。
- 对外 emit 语义完整的事件结果。

以搜索面板为例，组件内部维护筛选状态，点击搜索时 emit：

```ts
emit("search", {
  district,
  rent_mode,
  min_price,
  max_price,
  keyword,
  feature_flags,
});
```

父页面只处理：

```ts
async function onSearch(params: SearchHouseRequest) {
  const data = await searchHouses({ ...params, page: 1, page_size: 20 });
  cards.value = data.list.map(toListingCard);
}
```

禁止为了分层把搜索面板的输入框、开关、选中值全部抽到 `useDiscoverPage.ts`。

## 6. Services 使用规则

小程序前端默认不新增 `services/`。

只有满足以下条件之一，才允许新增 service：

- 一个业务动作需要组合多个 API。
- 多个页面复用同一套业务流程。
- 需要明确的缓存、重试、降级策略。

示例：

```text
可以：登录成功后写 token、拉 profile、写 profile cache。
不建议：单纯封装 searchHouses(data) 这种一对一 API 调用。
```

## 7. 样式组织

样式不长期堆在 `pages/` 下。

推荐：

```text
src/assets/css/discover/
src/assets/css/house-detail/
src/assets/css/login/
```

规则：

- 页面路由文件旁边不继续堆大量 `.scss`。
- 页面和组件样式统一放入 `src/assets/css/{module}/`。
- 跨页面公共样式放 `src/styles/`。
- 迁移时允许小步移动，不要求一次性全量改目录。

## 8. 当前迁移策略

不要一次性大重构。按接口和页面逐步迁移：

1. `api/house.ts`：先把 `house/search`、`house/public_detail` 改成 API 函数。
2. `models/listing.ts`：抽统一 HPD listing mapper。
3. `discover`：搜索面板状态回收到组件，页面只处理 API 调用和结果渲染。
4. `favorite/history`：复用 `models/listing.ts`。
5. `api/user.ts`、`api/favorite.ts`、`api/history.ts`：逐步补齐 request/response type 和调用函数。
6. 样式从 `pages/*` 迁到 `assets/css/{module}`。

`chat/send` 当前按既有旧接口保留，不纳入 HPD miniapp API 迁移。
