# Frontend Development Spec

适用于 `customer_mp` 前端工程。

技术栈：

- `Vue 3`
- `uni-app`
- `TypeScript`
- `SCSS`
- `uni.request`
- `Vite`

与 `shared-docs/` 中接口、字段、流程契约冲突时，以 `shared-docs/` 契约文档为准。与后端通用规范冲突时，以后端通用规范为准。

## 1. 总则

- 前端实现必须先对齐共享契约，再进入页面开发。
- 前端按页面驱动组织，不照搬后端分层。
- API 定义集中在 `api/`，组件 UI 状态尽量靠近组件。
- 只有跨页面复用、与 UI 无关的能力才抽到公共层。

## 2. 技术栈约定

- 框架统一使用 `Vue 3 + uni-app`。
- 语言统一使用 `TypeScript`。
- 样式统一使用 `SCSS`。
- 请求统一使用 `uni.request`。
- 构建统一使用 `Vite`。
- 环境变量统一使用 `VITE_` 前缀。

## 3. 目录结构约定

推荐目录：

```text
src/
  api/
  assets/
  components/
  config/
  models/
  pages/
  styles/
  utils/
  App.vue
  main.ts
  pages.json
```

目录职责：

- `api/`：接口定义层。维护 URL、请求方法、请求参数类型、响应类型和调用函数。
- `assets/css/`：页面和组件样式资源。
- `components/`：前端组件目录，按业务模块分子目录。
- `config/`：环境配置、公共配置。
- `models/`：后端 DTO 到前端 ViewModel 的转换，例如房源卡片、详情视图模型。
- `pages/`：路由页面入口，只放页面壳和页面级流程。
- `styles/`：公共样式、页面样式、组件样式。
- `utils/`：基础工具能力。

禁止：

- 页面直接拼接完整 URL。
- 页面直接发起 `uni.request(...)`。
- 重复实现同一套字段适配逻辑。
- `utils/` 依赖 `pages/` 或具体组件。
- 为了分层把组件内部 UI 状态强行抽到外部 service/composable。

## 4. 页面与路由约定

- 页面目录使用小写英文，按业务语义命名。
- 多单词页面使用短横线分隔，例如 `house-detail`、`profile-edit`。
- `pages.json` 负责页面注册、默认标题、`tabBar` 和全局页面配置。
- `pages.json` 中的 `navigationBarTitleText` 作为默认标题使用。
- 底部 `tabBar` 仅放一级主入口页面。

当前页面路径统一为：

- `pages/discover/discover`
- `pages/ai/ai`
- `pages/profile/profile`
- `pages/favorites/favorites`
- `pages/history/history`
- `pages/landlord/landlord`
- `pages/login/login`
- `pages/profile-edit/profile-edit`
- `pages/house-detail/house-detail`

## 5. 分层约定

前端分为五层：

```text
pages / components
  -> api
  -> models
  -> utils
```

### 5.1 API 层

位置：

```text
src/api/
```

职责：

- 定义接口短路径，例如 `"/house/search"`。
- 定义请求方式。当前业务接口默认 `POST`。
- 定义请求参数类型。
- 定义响应 `data` 类型。
- 暴露接口调用函数。

示例：

```ts
export interface SearchHouseRequest {
  district?: string;
  keyword?: string;
  min_price?: number;
  max_price?: number;
  feature_flags?: string[];
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

API 层不做：

- 页面 loading 状态。
- UI 展示文案。
- 组件交互状态。
- DTO 到 ViewModel 的展示映射。
- 本地缓存策略，除非该缓存是接口协议的一部分。

禁止只在 `api/` 中维护 URL 常量，再把 method、request type、response type 和调用函数散到其他目录。

### 5.2 Model / Mapper 层

位置：

```text
src/models/
```

职责：

- 后端 DTO 到页面 ViewModel 的转换。
- 统一空值展示文案、图片规范化、标签优先级、价格展示 fallback。

示例：

```ts
export function toListingCard(item: MiniappListingItem): ListingCardVM {
  return {
    id: item.listing_id,
    title: item.title || item.community_name || "房源信息",
    priceText: item.price_text || `${item.price}元/月`,
    cover: normalizeImageList(item.images)[0] || "",
  };
}
```

Model / Mapper 层不做：

- 发请求。
- 页面跳转。
- loading / toast / modal。
- 读取或写入页面状态。

同一类 DTO 的卡片映射只能有一份。收藏、足迹、找房列表如果展示同一种 HPD listing，应复用同一套 mapper。

### 5.3 Page 层

位置：

```text
src/pages/
```

职责：

- 小程序路由入口。
- 组合页面组件。
- 页面级流程，例如进入页面加载数据、搜索后刷新列表、点击卡片跳转详情。
- 调用 `api/` 和 `models/`。

Page 层不做：

- 直接调用 `uni.request`。
- 拼接完整 API URL。
- 维护复杂组件内部状态。
- 重复实现接口 DTO 字段映射。

### 5.4 Component 层

位置：

```text
src/components/{module}/
```

页面私有组件和公共组件统一放在 `components/` 下，按业务模块分目录。不要使用 `features` 作为目录名。

职责：

- UI 展示。
- 与组件界面强绑定的状态，例如表单输入、筛选展开、筛选项选择、重置按钮状态。
- 对外 emit 语义明确的事件或完整表单结果。

示例：

```ts
emit("search", {
  district,
  keyword,
  min_price,
  max_price,
  feature_flags,
});
```

Component 层一般不直接调业务 API。例外是高度独立的业务组件，例如上传组件；但仍不能拼接完整 URL 或绕过 API 层。

### 5.5 Utils / Infrastructure 层

位置：

```text
src/utils/
src/config/
```

职责：

- `config/env.ts`：维护环境域名与公共前缀。
- `utils/request.ts`：负责 URL 拼接、鉴权、错误处理、成功判定。
- `utils/http.ts`：封装通用 `get/post/...`。
- `utils/storageKeys.ts` 或类似文件：维护 storage key。
- 图片、登录态、格式化等真正通用能力。

Utils 层禁止依赖：

- `pages/`
- `components/`

### 5.6 Services 使用规则

默认不新增 `services/`。

只有出现真正跨接口业务编排时，才允许新增 service，例如：

- 同时调用多个 API 并合并结果。
- 有明确的缓存、重试、降级策略。
- 多页面复用同一套业务流程。

Service 不应成为 API 层和 Model 层的混合体。

## 6. 请求约定

### 6.1 URL 规则

- API 路径命名遵守 [../overview/project-spec.md](../overview/project-spec.md)。
- 基础域名来自环境变量，例如 `VITE_API_BASE_URL`。
- 公共前缀统一为 `/api/v1`。
- `api/*` 中统一写业务短路径，例如 `"/house/search"`。
- 业务短路径必须是 `/{模块}/{动作}`，例如 `"/house/search"`、`"/publish/create_centralized_project"`。

禁止：

- 在 `api/*` 中写完整域名。
- 在 `api/*` 中重复写 `/api/v1`。
- 在 `api/*` 中写 `"/miniapp/listing/search"` 这类多级业务路径。

### 6.2 请求方法

- 业务接口统一使用 `POST`。
- 未经共享文档确认，不新增 REST 风格 `GET/PUT/DELETE` 作为主业务接口标准。

### 6.3 鉴权

- 非登录、注册接口默认使用 `Authorization: Bearer <token>`。
- Token 统一从本地安全存储读取。
- Token 失效时必须清理本地登录态。

### 6.4 成功与失败

- 业务成功判定：`code === 0`
- HTTP 非 2xx：优先读取 `error`
- HTTP 2xx 但 `code !== 0`：按业务失败处理
- 错误提示统一由请求层处理

## 7. 接口字段与默认值约定

与后端通用规范保持一致：

- `0`、`0.0`、`""`、`null`、`undefined` 统一视为“未指定”
- 默认值不得解释为有意义业务状态

前端实现要求：

- 组装请求参数时，严格区分“未指定”和“有效值”
- 页面展示时不得直接输出默认值占位
- 查询参数默认值必须与共享契约一致

## 8. 命名约定

文件与目录：

- 页面目录：小写英文或短横线命名
- API 文件：按业务模块命名，如 `house.ts`、`user.ts`
- Model 文件：按展示模型命名，如 `listing.ts`、`profile.ts`

代码命名：

- 类型、接口：`PascalCase`
- 变量、函数：`camelCase`
- 常量：`UPPER_SNAKE_CASE` 或语义明确的常量对象

API 函数命名：

- 使用业务语义命名
- 避免使用 `getData`、`postInfo`、`doRequest` 等通用命名

## 9. 页面实现约定

- 页面只处理状态展示、用户输入、事件触发。
- 页面不直接消费后端原始结构，字段适配在 `models/` 中完成。
- 页面优先消费可直接渲染的数据结构。
- 页面内不重复处理通用网络错误与业务错误提示。
- 和组件界面强绑定的状态应留在组件内，由组件 emit 最终结果。

## 10. 样式约定

- 统一使用 `SCSS`
- 页面样式和组件样式不放在 `pages/` 目录长期堆积。
- 页面和组件样式统一放入 `src/assets/css/{module}/`。
- 跨页面公共样式放入 `src/styles/`。
- 小程序与 H5 均需保证可用
- 小程序页面尺寸优先使用 `rpx`

禁止：

- 大量内联样式
- 无明确理由混用多套尺寸单位
- 复制旧样式后不整理结构

## 11. 文档同步要求

- 新增页面、接口、字段映射规则且影响联调时，必须同步更新 `shared-docs`
- 接口路径、认证方式、默认值语义变化时，必须先更新共享文档，再改代码
- 前端实现规范以本文件为准，接口真值来源为：
  - `shared-docs/api/`
  - `shared-docs/schema/`
  - `shared-docs/changes/`
