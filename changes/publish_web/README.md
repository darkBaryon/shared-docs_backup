# 出房 Web 变更文档索引

本目录只记录出房 Web 端的阶段计划、coding plan 和阶段性决策。

## 目录结构

```text
changes/publish_web/
  README.md
  current-plan.md
  coding-plans/
  decisions/
  archive/
```

- `current-plan.md`：当前状态和下一步入口，保持短文档。
- `coding-plans/`：待 review 或已执行的具体编码计划，一个主题一个文件。
- `decisions/`：短决策记录，只放已经确认的方向。
- `archive/`：历史长文档归档，不作为日常阅读入口。

## 维护规则

- `current-plan.md` 只保留当前事实、入口链接和最近下一步，不继续堆完整历史。
- 新的 coding plan 必须落文件，review 通过前不进入代码实现。
- 长篇背景、旧计划和已完成细节放入 `archive/`。
- 接口字段和路径仍以 [../../api/publish.md](../../api/publish.md) 为准。
- 页面形态和前端长期约定沉淀到 [../../frontend/publish.md](../../frontend/publish.md)。
