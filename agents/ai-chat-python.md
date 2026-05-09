# AI Chat Python Agent

## 必读

1. [global.md](./global.md)
2. [../product/ai-chat.md](../product/ai-chat.md)
3. [../flows/ai-chat.md](../flows/ai-chat.md)
4. [../overview/system-map.md](../overview/system-map.md)

## 工作规则

- Python 后端负责 AI chat 相关能力，不是 Go 后端架构模板。
- Go 迁移时只参考 Python 的业务行为，不继承临时写法和技术边界。
- 需要读取房源、用户、会话等数据时，先确认数据来源属于 Go 后端、MongoDB 还是 Python 服务内部。
- 跨后端接口必须补充到 flows 或 api 文档。
