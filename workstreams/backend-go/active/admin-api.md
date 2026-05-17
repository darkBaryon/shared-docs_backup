# 管理端 API 实施计划

## 背景

后台管理端已开始实现。当前已落第一批 `admin_auth` 三个接口，管理端继续按独立端侧 HTTP 表达层和端侧应用服务推进。

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

## 当前进展

已完成：

1. `admin_auth` 接口契约已写入 `api/` 与 `backend/interfaces/`。
2. `admin_auth/login`、`admin_auth/session`、`admin_auth/logout` 已实现：
   - `handler/v1/admin/auth`
   - `service/admin/auth`
   - `repository/adm/*`
3. `admin_auth` 已接通：
   - `hs_adm_staff`
   - `hs_adm_staff_auth`
   - `hs_adm_staff_role`
   - `hs_adm_role`
   - `hs_adm_role_permission`
   - `hs_adm_permission`
   - `hs_adm_login_log`
   - Redis session store
4. `admin_auth` 基础测试已补：
   - handler 单测
   - service 单测
   - middleware 单测
   - `tests/integration/admin_auth` Mongo + Redis 集成测试骨架
5. `staff/create` 与 `staff/list` 接口级文档已补：
   - `backend/interfaces/admin/staff_create.md`
   - `backend/interfaces/admin/staff_list.md`
   - `api/admin.md` staff 创建 / 列表字段、权限和错误提示已同步
6. `staff/create` 已实现：
   - `handler/v1/admin/staff`
   - `service/admin/staff`
   - `repository/adm` staff / staff_auth / staff_role 写能力
   - `staff.edit` 权限 middleware 校验
   - handler / service / middleware 单测
7. `staff/list` 已实现：
   - `handler/v1/admin/staff`
   - `service/admin/staff`
   - `repository/adm` staff 分页查询、staff_role 批量查询、staff_auth 登录摘要查询
   - `staff.view` 权限 middleware 校验
   - handler / service 单测
8. `staff/detail` 已实现：
   - `handler/v1/admin/staff`
   - `service/admin/staff`
   - `repository/adm` staff 单条详情查询、staff_role / role / staff_auth 摘要查询
   - `staff.view` 权限 middleware 校验
   - handler / service 单测
9. `staff/update` 已实现：
   - `handler/v1/admin/staff`
   - `service/admin/staff`
   - `repository/adm` staff 字段更新、staff_role 角色替换
   - `staff.edit` 权限 middleware 校验
   - handler / service 单测
10. `staff/disable` 已实现：
   - `handler/v1/admin/staff`
   - `service/admin/staff`
   - `repository/adm` staff 软禁用
   - `staff.edit` 权限 middleware 校验
   - handler / service 单测
11. `role/list`、`role/detail`、`role/create`、`role/update` 已实现：
   - `handler/v1/admin/role`
   - `service/admin/role`
   - `repository/adm` role / permission / role_permission 读写能力
   - `role.view/role.edit` 权限 middleware 校验
   - 角色权限变更后失效已分配员工 admin token
   - handler / service 单测
12. `provider/list`、`provider/detail`、`provider/create`、`provider/update`、`provider/disable` 已实现：
   - `handler/v1/admin/provider`
   - `service/admin/provider`
   - `repository/landlord` landlord / landlord_auth 后台账号读写能力
   - `provider.view/provider.edit` 权限 middleware 校验
   - 创建采用非事务补偿清理，修改手机号和禁用后失效 publish token
   - handler / service 单测

## 下一步

1. 进入 `house` 模块前先补平台房源查询、详情和运营动作需求文档。
2. 再按一接口一文档补 `house` 模块接口契约。
3. 后续如需改密，再单独进入 `reset_password` 或 `staff/reset_password`。
