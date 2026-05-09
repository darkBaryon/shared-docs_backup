# AI Chat

AI chat 是 AI 找房对话能力。

## 当前关系

- Python 后端 `ai_house` 保留 AI chat 能力。
- Go 后端 `house-manager` 负责 auth、主数据、发房和后续管理能力。
- Go 迁移参考 Python 的业务行为，不继承 Python 技术架构。

## 待明确

- AI chat 读取房源展示数据的来源。
- AI chat 与 Go 后端之间是否需要内部 API。
- 用户会话、线索和跟进数据的归属边界。
