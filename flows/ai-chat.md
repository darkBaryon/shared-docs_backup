# AI Chat Flow

AI chat 流程尚未完整定稿。

## 当前关系

```text
小程序 AI 入口
  -> Python AI chat
    -> 读取或引用房源、用户、线索数据
```

## 待明确

- AI chat 查询房源时是否调用 Go 后端 API。
- AI chat 是否直接读取 HPD 展示层数据。
- AI chat 产生的线索和跟进数据归属。
