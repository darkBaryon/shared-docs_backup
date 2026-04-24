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
- 页面层负责展示与交互，业务规则下沉到 `services/`，请求规则收口到 `utils/request.ts`。

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
  components/
  config/
  pages/
  services/
  utils/
  App.vue
  main.ts
  pages.json
```

目录职责：

- `api/`：接口路径常量。
- `components/`：跨页面复用组件。
- `config/`：环境配置、公共配置。
- `pages/`：页面文件。
- `services/`：业务服务层。
- `utils/`：基础工具能力。

禁止：

- 页面直接拼接完整 URL。
- 页面直接发起 `uni.request(...)`。
- 重复实现同一套字段适配逻辑。

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

## 5. 请求层约定

请求层分为四层：

- `config/env.ts`：维护环境域名与公共前缀。
- `api/endpoints.ts`：维护业务短路径。
- `utils/request.ts`：负责 URL 拼接、鉴权、错误处理、成功判定。
- `services/*.ts`：封装业务语义与字段适配。

### 5.1 URL 规则

- 基础域名来自环境变量，例如 `VITE_API_BASE_URL`。
- 公共前缀统一为 `/api/v1`。
- `endpoints.ts` 中统一写业务短路径，例如 `"/house/search"`。

禁止：

- 在 `endpoints.ts` 中写完整域名。
- 在 `endpoints.ts` 中重复写 `/api/v1`。

### 5.2 请求方法

- 业务接口统一使用 `POST`。
- 未经共享文档确认，不新增 REST 风格 `GET/PUT/DELETE` 作为主业务接口标准。

### 5.3 鉴权

- 非登录、注册接口默认使用 `Authorization: Bearer <token>`。
- Token 统一从本地安全存储读取。
- Token 失效时必须清理本地登录态。

### 5.4 成功与失败

- 业务成功判定：`code === 0`
- HTTP 非 2xx：优先读取 `detail`，其次 `message`、`msg`
- HTTP 2xx 但 `code !== 0`：按业务失败处理
- 错误提示统一由请求层处理

## 6. 接口字段与默认值约定

与后端通用规范保持一致：

- `0`、`0.0`、`""`、`null`、`undefined` 统一视为“未指定”
- 默认值不得解释为有意义业务状态

前端实现要求：

- 组装请求参数时，严格区分“未指定”和“有效值”
- 页面展示时不得直接输出默认值占位
- 查询参数默认值必须与共享契约一致

## 7. 命名约定

文件与目录：

- 页面目录：小写英文或短横线命名
- 服务文件：按业务模块命名，如 `house.ts`、`user.ts`
- 接口常量文件：`endpoints.ts`

代码命名：

- 类型、接口：`PascalCase`
- 变量、函数：`camelCase`
- 常量：`UPPER_SNAKE_CASE` 或语义明确的常量对象

服务函数命名：

- 使用业务语义命名
- 避免使用 `getData`、`postInfo`、`doRequest` 等通用命名

## 8. 页面实现约定

- 页面只处理状态展示、用户输入、事件触发。
- 页面不直接消费后端原始结构，字段适配在 `services/` 中完成。
- 页面优先消费可直接渲染的数据结构。
- 页面内不重复处理通用网络错误与业务错误提示。

## 9. 样式约定

- 统一使用 `SCSS`
- 页面样式默认局部管理，共享样式抽到公共样式文件
- 小程序与 H5 均需保证可用
- 小程序页面尺寸优先使用 `rpx`

禁止：

- 大量内联样式
- 无明确理由混用多套尺寸单位
- 复制旧样式后不整理结构

## 10. 文档同步要求

- 新增页面、接口、字段映射规则且影响联调时，必须同步更新 `shared-docs`
- 接口路径、认证方式、默认值语义变化时，必须先更新共享文档，再改代码
- 前端实现规范以本文件为准，接口真值来源为：
  - `shared-docs/api/`
  - `shared-docs/schemas/`
  - `shared-docs/contracts/`
