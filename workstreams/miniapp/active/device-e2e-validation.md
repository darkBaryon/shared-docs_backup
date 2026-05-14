# 真机 E2E 验证计划

## 背景

小程序前端字段对齐已经完成，需要用真机和 Go 后端测试数据跑完整链路。

## 验证链路

1. 微信登录。
2. 未注册时微信手机号注册。
3. 找房列表展示 HPD 房源。
4. 房源详情展示 HPD 图片、价格、标签、基础信息。
5. 收藏 / 取消收藏。
6. 收藏列表展示。
7. 进入详情写入足迹。
8. 足迹列表展示。
9. 个人资料读取和更新。

## 自动检查

以下范围内不应再出现旧房源主链路字段：

```text
src/services/favorite
src/services/history
src/pages/favorites
src/pages/history
```

禁止字段：

```text
house_id
_id fallback
area/location 作为房源展示字段
room_count/hall_count
images: string[] raw contract
limit 作为分页请求
favorites/history 作为列表响应 envelope
```

允许例外：

- UI 文案中的“区域”等中文展示词。
- 非房源主链路上下文中的普通状态字段。
- `src/services/chat` 和 `src/pages/ai`：本期明确保持旧接口。

## 后端日志期望

E2E 过程中应出现：

```text
POST /api/v1/auth/wechat_login
POST /api/v1/auth/wechat_register
POST /api/v1/house/search
POST /api/v1/house/public_detail
POST /api/v1/favorite/add
POST /api/v1/favorite/remove
POST /api/v1/favorite/list
POST /api/v1/history/add
POST /api/v1/history/list
POST /api/v1/user/profile
POST /api/v1/user/update_profile
POST /api/v1/user/dashboard
```

不应出现：

```text
house_id
POST /api/v1/house/detail
POST /api/v1/house/list
POST /api/v1/house/create
POST /api/v1/house/status
```

## 完成后更新

- [../status.md](../status.md)
- 相关 review report 写入 `../reviews/`
