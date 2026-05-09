# Miniapp Search Flow

当前小程序找房流程尚未完整落地。

## 目标形态

```text
小程序
  -> Go 后端房源查询 API
    -> HPD 展示层数据
      -> MongoDB
```

HPD 未完成前，不应让小程序直接依赖 HMD 主数据结构。
