# AI 找房数据库设计 V4

按模块拆分：
- `modules/common-fields.md`
- `modules/account-and-identity.md`
- `modules/house-master-data.md`
- `modules/house-publish-and-display.md`
- `modules/house-audit-and-change.md`
- `modules/user-activity.md`
- `modules/lead-and-followup.md`
- `modules/ops-and-marketing.md`
- `modules/redis-design.md`
- `modules/todo-and-open-questions.md`

## 当前进度

### 已完成
- `modules/common-fields.md`
- `modules/account-and-identity.md`
- `modules/house-master-data.md`
- `modules/house-publish-and-display.md`

### 待检查
- `modules/house-audit-and-change.md`

### 未完成
- `modules/user-activity.md`
- `modules/lead-and-followup.md`
- `modules/ops-and-marketing.md`
- `modules/redis-design.md`
- `modules/todo-and-open-questions.md`（持续维护）

## 命名规则

集合命名格式：`hs_{模块简称}_{业务实体}`

- `hs`：House 项目前缀（固定）
- `{模块简称}`：业务模块缩写
- `{业务实体}`：实体名

## 模块缩写全称

| 缩写 | 全称 | 含义 |
| --- | --- | --- |
| `usr` | user | C 端用户域 |
| `adm` | administration | 后台管理域（员工/角色/权限） |
| `hmd` | house master data | 房源主数据域 |
| `hpd` | house publish display | 房源发布与展示域 |
| `hac` | house audit change | 房源审核与变更域 |
| `lead` | lead | 客户与跟进域 |
| `ops` | operations | 运营营销域 |

## 通用规范

- 主键：`_id:ObjectId`
- 外键：`xxx_id:ObjectId`
- 公共字段：`created_at` `updated_at` `status` `version`
- 状态值：`0` 未指定，`1..99` 有效，`-1..-99` 无效/删除
