# 智小窝找房后端数据库结构设计（V5，非兼容重建版）

## 1. 目标

直接按租客检索场景重建房源结构，不做旧字段兼容，支持：
- 常见需求快捷按钮（近地铁、押一付一、可养宠物、整租两居等）
- 高级筛选面板（区域、价格、租住方式、户型、付款方式、标签）

---

## 2. `hs_hs_house` 建议字段（V5）

| 字段名 | 类型 | 必填 | 说明 |
|---|---|---|---|
| `_id` | String | 是 | MongoDB 主键 |
| `owner_id` | String | 是 | 房东 ID |
| `area` | String | 是 | 行政区（南山/福田/宝安/罗湖） |
| `location` | String | 是 | 小区/街道 |
| `price` | Int | 是 | 月租金（整数） |
| `status` | Int | 是 | `1`待租 / `2`已租 / `0`下架 |
| `images` | List[String] | 否 | 房源图片 URL |
| `created_at` | Int64 | 是 | 创建时间戳（秒） |
| `updated_at` | Int64 | 是 | 更新时间戳（秒） |
| `rent_mode` | String | 否 | `whole`（整租）/`shared`（合租） |
| `room_count` | Int | 否 | 室数量 |
| `hall_count` | Int | 否 | 厅数量 |
| `bathroom_count` | Int | 否 | 卫数量 |
| `payment_cycle` | String | 否 | `押一付一` / `押一付三` / `月付` / `季付` |
| `near_subway` | Bool | 否 | 是否近地铁 |
| `walk_to_subway_min` | Int | 否 | 步行至地铁分钟数 |
| `pet_friendly` | Bool | 否 | 是否可养宠物 |
| `has_elevator` | Bool | 否 | 是否有电梯 |
| `cooking_allowed` | Bool | 否 | 是否可做饭 |
| `furnished_level` | String | 否 | `none`/`basic`/`full`（用于表达拎包入住程度） |
| `tags` | List[String] | 否 | 展示标签（推荐从结构化字段计算生成，不作为主筛选字段） |

---

## 3. 设计原则

1. **结构化筛选优先**：所有搜索关键条件都落到独立字段。  
2. **展示与检索分离**：`tags` 用于展示，不作为唯一检索依据。  
3. **规范输入**：`area/rent_mode/payment_cycle/furnished_level` 使用受控枚举值。

---

## 4. 推荐索引

### 4.1 房东管理
- `{ owner_id: 1, status: 1 }`

### 4.2 公开搜索主链路
- `{ status: 1, area: 1, price: 1 }`
- `{ status: 1, rent_mode: 1, room_count: 1, price: 1 }`
- `{ status: 1, near_subway: 1, walk_to_subway_min: 1, price: 1 }`
- `{ status: 1, payment_cycle: 1, price: 1 }`
- `{ status: 1, tags: 1 }`（multikey）

---

## 5. 对前端能力的直接收益

- “常见需求按钮”不再依赖模糊匹配，可稳定命中：
  - 近地铁 -> `near_subway=true` 或 `tags` 包含近地铁
  - 押一付一 -> `payment_cycle=押一付一`
  - 可养宠物 -> `pet_friendly=true`
  - 整租两居 -> `rent_mode=whole` + `room_count>=2`
- “更多筛选”可直接落地为结构化查询，不会出现“点了筛选却搜不到”的体验问题。

---
