# Admin Interface Docs

本目录按“一个接口一份文档”维护管理后台后端实现说明。

管理后台是 `admin` terminal，服务对象是平台员工，不复用 `publish` 房东端 service。

## `admin_auth` 统一约束

- 登录主体固定是 `staff`。
- session principal 固定是 `terminal=admin`、`principal_type=staff`。
- 登录手机号来自 `hs_adm_staff.phone`。
- 密码 hash 来自 `hs_adm_staff_auth.password_hash`。
- 角色与权限来自：
  - `hs_adm_staff_role`
  - `hs_adm_role`
  - `hs_adm_role_permission`
  - `hs_adm_permission`
- 登录态写入 Redis session store，token 是 opaque token，不是 JWT。
- 不复用 `miniapp` 或 `publish` 的 auth service，不读取 `hs_usr_*`、`hs_lld_*` 作为 admin 登录数据源。

## `admin_auth`

- [POST /api/v1/admin_auth/login](./admin_auth_login.md)
- [POST /api/v1/admin_auth/session](./admin_auth_session.md)
- [POST /api/v1/admin_auth/logout](./admin_auth_logout.md)
- [POST /api/v1/admin_auth/reset_password](./admin_auth_reset_password.md)

## `staff`

- [POST /api/v1/staff/create](./staff_create.md)
- [POST /api/v1/staff/list](./staff_list.md)
- [POST /api/v1/staff/detail](./staff_detail.md)
- [POST /api/v1/staff/update](./staff_update.md)
- [POST /api/v1/staff/disable](./staff_disable.md)

## `role`

- [POST /api/v1/role/list](./role_list.md)
- [POST /api/v1/role/detail](./role_detail.md)
- [POST /api/v1/role/create](./role_create.md)
- [POST /api/v1/role/update](./role_update.md)

## `provider`

- [POST /api/v1/provider/list](./provider_list.md)
- [POST /api/v1/provider/detail](./provider_detail.md)
- [POST /api/v1/provider/create](./provider_create.md)
- [POST /api/v1/provider/update](./provider_update.md)
- [POST /api/v1/provider/disable](./provider_disable.md)

## `house`

- [POST /api/v1/house/list](./house_list.md)
- [POST /api/v1/house/detail](./house_detail.md)

## 当前实现顺序

首批实现：

1. `admin_auth/login`
2. `admin_auth/session`
3. `admin_auth/logout`
4. `staff/create`
5. `staff/list`
6. `staff/detail`
7. `staff/update`
8. `staff/disable`
9. `role/list`
10. `role/detail`
11. `role/create`
12. `role/update`
13. `provider/list`
14. `provider/detail`
15. `provider/create`
16. `provider/update`
17. `provider/disable`
18. `house/list`
19. `house/detail`

后移：

- `admin_auth/reset_password`
- `house/export`
