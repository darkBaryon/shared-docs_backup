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

小程序 auth 不作为 publish/admin 的通用登录入口。正式 publish auth 契约确认前，出房 Web 端只需要：

- 请求层支持 Bearer Token。
- Token 从本地存储读取。
- 开发联调时允许提供临时 Token 设置入口。
- 不复用小程序微信登录。

## 文档维护

- 当前状态和下一步维护在 [../workstreams/publish-web/status.md](../workstreams/publish-web/status.md)。
- 接口路径、请求字段、响应字段维护在 [../api/publish.md](../api/publish.md)。
- 字段、枚举、索引维护在 [../schema/db-design/v4/modules/house-master-data.md](../schema/db-design/v4/modules/house-master-data.md)。
