# 智小窝找房：个人主页与双角色模式设计方案

## 1. 目标

在不重写现有“找房页 / 咨询页”的前提下，支持同一账号在小程序内切换两种身份：
- `tenant`（租客）
- `landlord`（房东）

核心原则：**同壳双模式**。  
即页面入口不变、路由不变，按 `role` 切换页面内容与接口调用。

---

## 2. 产品结构

### 2.1 Tab 结构（保持不变）
- 找房（`discover`）
- 咨询（`ai`）
- 我的（`profile`）

### 2.2 角色下的页面语义

#### 租客模式（tenant）
- 找房页：公开房源检索（现有）
- 咨询页：找房咨询（现有）
- 个人主页：收藏、浏览记录、偏好设置

#### 房东模式（landlord）
- 找房页：房东工作台（我的房源入口、发布入口、房源状态摘要）
- 咨询页：运营助手（租客咨询摘要、常见问答、AI文案辅助）
- 个人主页：账号信息、房东统计、角色切换、退出登录

---

## 3. 前端架构设计

## 3.1 全局状态

在 `app.ts` 的 `globalData` 新增：
- `role: 'tenant' | 'landlord'`
- `token: string`
- `profile: UserProfile | null`

并通过 `wx.setStorageSync('role', role)` 持久化角色。

## 3.2 Profile 页职责

`profile` 页作为角色中心，负责：
1. 拉取个人信息（昵称、头像、联系方式、当前角色）
2. 展示角色切换控件（`tenant <-> landlord`）
3. 切换后更新全局状态并触发页面刷新

建议增加一个轻量全局事件（或 `getApp().globalData` 轮询）：
- `role_changed`

`discover` / `ai` 页面在 `onShow` 中读取最新 `role` 并重绘。

## 3.3 页面改造策略（不重写）

### discover
- 保留现有租客 UI
- 新增 `wx:if="{{role === 'tenant'}}"` 与 `wx:else` 分支
- 房东分支优先放“我的房源操作卡”：
  - 新建房源
  - 我的房源列表
  - 待租/已租/下架统计

### ai
- 保留现有租客咨询 UI
- 新增房东模式 UI（先做 MVP）：
  - 示例：发布文案助手、租客常见问题回复模板
  - 后续再接业务会话流

### profile
- 作为唯一角色切换入口
- 顶部显示当前身份徽标
- 中部显示身份对应快捷入口
- 底部保留通用账号设置

---

## 4. 后端接口设计（MVP）

统一风格：`POST /api/v1/{module}/{action}`

## 4.1 用户模块

### `POST /api/v1/user/profile`
返回当前用户资料：
- `user_id`
- `role`
- `nickname`
- `avatar`
- `contact`

### `POST /api/v1/user/profile_update`
更新资料：
- `nickname`
- `avatar`
- `contact`

### `POST /api/v1/user/switch_role`
请求：
- `role: tenant | landlord`

响应：
- `role`
- `switched_at`

## 4.2 统计模块（个人主页聚合）

### `POST /api/v1/user/dashboard`
按角色返回不同字段：

租客：
- `favorite_count`
- `history_count`
- `session_count`

房东：
- `house_total`
- `house_on_rent`
- `house_rented`
- `house_offline`

---

## 5. 数据模型建议

`hs_usr_user` 增补/确认字段：
- `role`（`tenant` / `landlord`）
- `nickname`
- `avatar`
- `contact`
- `global_preferences`
- `last_active`

说明：
- 一个用户同一时刻只有一个“当前角色”
- 如需同时管理双角色扩展，可后续加 `roles: string[]`

---

## 6. 交互流程

## 6.1 首次进入
1. 登录拿 token
2. 调 `/user/profile` 拿 role（默认 tenant）
3. 写入 `globalData.role` 与本地缓存

## 6.2 角色切换
1. 用户在 `profile` 点击切换
2. 调 `/user/switch_role`
3. 成功后更新 `globalData.role`
4. `discover` / `ai` onShow 自动按新角色渲染

---

## 7. 实施顺序（建议）

1. 后端先补 `user/profile` + `user/switch_role` + `user/dashboard`  
2. 前端先改 `profile` 页（可切换角色）  
3. 前端给 `discover` / `ai` 加 role 分支渲染  
4. 最后补房东模式下的具体功能入口（房源管理、运营助手）

---

## 8. 验收标准（MVP）

- 可在个人主页看到当前角色并成功切换
- 切换后返回找房/咨询页内容跟随变化
- 重启小程序后仍保持上次角色
- 角色切换失败有 Toast 提示，且不污染本地状态

