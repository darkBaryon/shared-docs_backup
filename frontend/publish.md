# Frontend Publish

发房系统前端以 [../api/publish.md](../api/publish.md) 为接口依据。

当前开发计划见 [../workstreams/publish-web/status.md](../workstreams/publish-web/status.md)。

## 当前后端状态

- 发房端后端第一期 HMD 链路已通过真实服务联调。
- 前端可以开始接入 HMD 录入、详情、列表、更新和房态流转。
- 当前接口已经走 Redis Session 鉴权，联调时需要 `Authorization: Bearer <token>`。
- HPD 小程序 projector 已接入，暂时仍不要把 HMD 结果当成小程序展示层 API 数据。

## 约定

- 技术栈与后台 Web 端保持一致：Vue 3、Vite、TypeScript、SCSS、Element Plus、Pinia、Vue Router、Axios。
- API 路径使用 `POST /api/v1/{业务module}/{action}`；`publish` 是端侧系统名，不占用 API module 位。
- 表单枚举值以 schema 为准。
- 分散式房间当前不提供房型选择。
- 发房系统和后台管理系统不是同一个前端。
- 当前出房 Web 端仓库未建立，临时在后台 Web 仓库新分支内开发；新仓库初始化后直接迁移代码并删除临时分支。
- 目录结构优先参考后台 Web `feature_20260312` 分支的轻量架构。
- API module 叫 `publish` 不代表前端目录必须叫 `publish`。

## 目录约定

推荐参考 0312 分支的轻量后台架构：

```text
src/
  main.ts
  App.vue
  router/
    index.ts
  views/
    layout/
    home/
    centralized/
    decentralized/
  api/
  model/
  utils/
  stores/
  assets/css/
```

约束：

- 不使用 `src/publish/` 作为最终目录结构。
- 页面按 `views/centralized/*` 和 `views/decentralized/*` 组织。
- API 和 model 按业务对象拆分，例如 `api/centralizedProject.ts`、`model/centralizedProject.ts`。
- 页面不直接拼接完整 URL，不直接调用 Axios。
- API 层维护业务短路径、请求类型、响应类型和调用函数。

## 第一期开口范围

第一期只做 HMD 录入与维护：

- 集中式项目
- 集中式楼栋
- 集中式房型
- 集中式房间
- 分散式小区
- 分散式房间

暂不做：

- 后台审核和运营管理。
- 小程序 C 端页面。
- HPD 发布内容编辑。
- 上架审核流。
- 房东端协作能力。
- 文件上传服务。

## 鉴权

publish 路由当前需要：

```text
Authorization: Bearer <token>
```

小程序 auth 不作为 publish/admin 的通用登录入口。正式出房 Web 流程使用 publish auth：

```text
POST /api/v1/publish_auth/login
POST /api/v1/publish_auth/session
POST /api/v1/publish_auth/logout
```

前端约束：

- 登录页或登录面板调用 `publish_auth/login` 获取 token。
- App 初始化时调用 `publish_auth/session` 恢复登录态。
- 请求层只设置 `Authorization: Bearer <token>`，不解析 token。
- 401 或业务码 `10002` 时清理本地 token 并回到未登录态。
- 不复用小程序微信登录。
- 开发 token 面板只允许在 `VITE_ENABLE_DEV_TOKEN_PANEL=true` 时出现；未开启时不能渲染入口，也不能作为正式登录流程。

## 数据作用域

出房 Web 不在前端决定账号数据范围。前端只提交 city/district/name 等业务筛选字段，后端从 publish session principal 执行数据作用域校验。

禁止在业务请求中提交：

```text
user_id
staff_id
owner
owner_id
owner_phone
maintainer_staff_id
service_staff_id
```

约束：

- HMD 不加 owner；前端 model、form、query 不新增 HMD owner 字段。
- 项目/小区列表在 session 有效后自动加载；搜索条件只能缩小当前账号已授权范围，不能扩大权限。
- 楼栋、房型、房间列表以接口返回为准，不做本地权限推断。

## 文档维护

- 当前状态和下一步维护在 [../workstreams/publish-web/status.md](../workstreams/publish-web/status.md)。
- 接口路径、请求字段、响应字段维护在 [../api/publish.md](../api/publish.md)。
- 字段、枚举、索引维护在 [../schema/db-design/v4/modules/house-master-data.md](../schema/db-design/v4/modules/house-master-data.md)。
