# V4 待定与开放问题

本文档仅保留当前仍未定稿的问题。每项都附定位信息（相对路径 + 行号）。

## 1. contact_qr_code 字段语义

- 问题：`contact_qr_code` 的真实语义不清（是长期活码、临时二维码 URL，还是其他渠道值）。
- 当前临时处理：字段保留，备注“含义待确认”。
- 定位：`modules/account-and-identity.md:82`

## 2. role 与 permission 清单待定

- role 待定：最终有哪些角色（`role_code`/`role_name`）。
- 定位：`modules/account-and-identity.md:182-190`

- permission 待定：最终有哪些权限点（`permission_code`/`permission_name`）。
- 定位：`modules/account-and-identity.md:192-201`
